# Cloud Computing Final Project
Kelompok C3
- Fina Keiza Arismana (5027211028)
- Reynold Putra Merdeka (5027211034)
- Rifqi Akhmad Maulana (5027211035)
- Wisnuyasha Faizal (5027211036)
- Ahnaf Musyaffa (5027211038)
- Maria Teresia Elvara Bumbungan (5027211042)


## Problem

Anda adalah seorang lulusan Teknologi Informasi, sebagai ahli IT, salah satu kemampuan yang harus dimiliki adalah **Keampuan merancang, membangun, mengelola aplikasi berbasis komputer menggunakan layanan awan untuk memenuhi kebutuhan organisasi.**(menurut kurikulum IT ITS 2023 😙) 

Pada suatu saat teman anda ingin mengajak anda memulai bisnis di bidang digital marketing, anda diberikan sebuah aplikasi berbasis API File: [app.py](/app.py)

Kemudian anda diminta untuk mendesain arsitektur cloud yang sesuai dengan kebutuhan aplikasi tersebut. Apabila dana maksimal yang diberikan adalah 1 juta rupiah per bulan (65 US$) konfigurasi cloud terbaik seperti apa yang bisa dibuat?



## Endpoints dan Database:
### Endpoints
1. **Get All Orders**
   - **Endpoint:** `GET /orders`
   - **Description:** Retrieve a list of all orders.
   - **Response:**
     ```json
     {
       "orders": [
         {
            "_id": "order_id_1", "product": "Product1", "quantity": 5, "customer_name": "John Doe", "customer_address": "123 Main St"
            },
         {
            "_id": "order_id_2", "product": "Product2", "quantity": 3, "customer_name": "Jane Smith", "customer_address": "456 Oak St"
        },
         // ...
       ]
     }
     ```
2. **Create a New Order**
   - **Endpoint:** `POST /orders`
   - **Description:** Create a new order.
   - **Request:**
     ```json
     {
       "product": "ProductY",
       "quantity": 2,
       "customer_name": "Bob Anderson",
       "customer_address": "101 Pine St"
     }
     ```
   - **Response:**
     ```json
     {
       "message": "Order created successfully",
       "order": {
            "_id": "new_order_id", "product": "ProductY", "quantity": 2, "customer_name": "Bob Anderson", "customer_address": "101 Pine St"
        }
     }
     ```
### Database
 - **Database:** `orders_db`
- **Connection URI:** `mongodb://localhost:27017/orders_db`

## Analisis
- Program yang dijalankan adalah REST API dengan payload yang dikirim cukup kecil/simple sehingga tidak membutuhkan komputasi lebih.
- Database yang digunakan adalah MONGO DB dengan skema satu collection saja.
- Komputasi yang dibutuhkan cukup rendah sehingga tidak membutuh spec machine yang tinggi.
- Hipotesis : Menggunakan spec machine rendah namun dapat melakukan scale out secara horizontal untuk mendukung concurrent request
## Arsitektur
Untuk membangun arsitektur microservices dengan topologi seperti di atas digunakan platform Microsoft Azure dengan Student Subscription senilai $100. 

Dikarenakan sistem billing Microsoft Azure menggunakan sistem **pay-as-you-go**, dan mengingat bahwa VM independen memiliki harga yang cukup mahal, maka diputuskan untuk menggunakan Container App yang ada di Azure.

Adapun beberapa benefit yang didapatkan jika menggunakan Container App dari Azure adalah:
- Penggunaan / utilisasi resource yang lebih efisien.
- Container Apps Azure memiliki integrasi dengan Kubernetes, dengan kata lain dapat di-upscale secara otomatis (membuat replika container dengan scale-rule yang telah ditentukan)
- Lebih murah dibandingkan dengan VM

## Langkah Implementasi
- Resource group berguna untuk grouping sumberdaya yang akan digunakan. Dalam kasus ini diberi nama tka-c3-fp.
- Buat Container Registry untuk menyimpan hasil image dari Flask
- Buat virtual network
- Buat Container App Environment
  - Virtual Network diekspos secara external sehingga container app dapat diakses secara public menggunakan protocol tcp dan http
- Buat Container App untuk Mongo DB dengan ENV sebagai berikut
  - MONGO_INITDB_ROOT_USERNAME={username_mongo}
  - MONGO_INITDB_ROOT_PASSWORD={password_mongo}
- Buat ContainerApp untuk Flask dengan ENV sebagai berikut
  - MONGO_URI=mongodb://{username_mongo}:{password_mongo}@{container_app_mongo}/db
  - PORT=5000
  - HOST=0.0.0.0
- Atur scale rule pada Mongo
```bash
az containerapp update \
  --name mongo-open \
  --resource-group tka-c3-fp \
  --environment tka-env-open \
  --min-replicas 1 \
  --max-replicas 5 \
  --scale-rule-name azure-tcp-rule \
  --scale-rule-type tcp \
  --scale-rule-tcp-concurrency 1

```
- Atur svale rule pada Flask    
```bash
az containerapp update \
  --name flask-open \
  --resource-group tka-c3-fp \
  --environment tka-env-open \
  --min-replicas 1 \
  --max-replicas 35 \
  --scale-rule-name azure-http-rule \
  --scale-rule-type http \
  --scale-rule-http-concurrency 1
```
- Buat volume mount untuk mongo

## Hasil Pengujian Enpoint

## Load Testing
