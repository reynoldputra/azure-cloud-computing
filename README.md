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

![tka cloud architecture drawio](https://github.com/reynoldputra/azure-cloud-computing/assets/112788819/050c3941-5775-439a-9ce6-e14ff7e5c7df)

Untuk membangun arsitektur microservices dengan topologi seperti di atas digunakan platform Microsoft Azure dengan Student Subscription senilai $100. 

Dikarenakan sistem billing Microsoft Azure menggunakan sistem **pay-as-you-go**, dan mengingat bahwa VM independen memiliki harga yang cukup mahal, maka diputuskan untuk menggunakan Container App yang ada di Azure.

Adapun beberapa benefit yang didapatkan jika menggunakan Container App dari Azure adalah:
- Penggunaan / utilisasi resource yang lebih efisien.
- Container Apps Azure memiliki integrasi dengan Kubernetes, dengan kata lain dapat di-upscale secara otomatis (membuat replika container dengan scale-rule yang telah ditentukan)
- Lebih murah dibandingkan dengan VM


## Langkah Implementasi
- Resource group berguna untuk grouping sumberdaya yang akan digunakan. Dalam kasus ini diberi nama tka-c3-fp.
![image](https://github.com/reynoldputra/azure-cloud-computing/assets/87769109/c8072e36-d707-4534-8707-e4656d400c60)
- Buat Container Registry untuk menyimpan hasil image dari Flask
![image](https://github.com/reynoldputra/azure-cloud-computing/assets/87769109/1a6f76b2-b6ac-431b-b98c-36507f9b8f13)
- Checkbox "Admin User" pada Access Keys Container Register agar repository dapat dipanggil
![image](https://github.com/reynoldputra/azure-cloud-computing/assets/87769109/718b0261-3bbf-44c3-929b-ed2e16c3fb61)
- Buat Virtual network
![image](https://github.com/reynoldputra/azure-cloud-computing/assets/87769109/037273ea-3ce6-4ff2-8f24-ee08f9fbf15a)
- Buat Container App untuk Mongo DB dan Container app environment dengan ENV sebagai berikut
  - MONGO_INITDB_ROOT_USERNAME={username_mongo}
  - MONGO_INITDB_ROOT_PASSWORD={password_mongo}
![image](https://github.com/reynoldputra/azure-cloud-computing/assets/87769109/eb2e06ae-99a1-43f2-affd-2040f27dffbc)
- Virtual Network diekspos secara external sehingga container app dapat diakses secara public menggunakan protocol tcp dan http
![image](https://github.com/reynoldputra/azure-cloud-computing/assets/87769109/78a980a9-698b-4d88-a3a1-ad597c5ba663)
- Buat ContainerApp untuk Flask dengan ENV sebagai berikut
  - MONGO_URI=mongodb://{username_mongo}:{password_mongo}@{container_app_mongo}/db
  - PORT=5000
  - HOST=0.0.0.0
![image](https://github.com/reynoldputra/azure-cloud-computing/assets/87769109/847ef521-78b0-4c33-a4e7-289bc2aed5f4)

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
Storage mount dibuat agar azure dapat menyimpan semua data dari server container. Ini dibuat agar data container tidak hilang ketika server mati.
Pertama, buat storage account.

![Screenshot 2023-12-12 220642](https://github.com/reynoldputra/azure-cloud-computing/assets/100693456/fafd25b6-920f-4fee-947b-63dd80218282)

Ini mengarah ke resource group tka-c3 yang telah dibuat, dengan instance details standard performance dan LRS redundancy.

Selanjutnya, Tambahkan file share yang terletak didalam storage account.
![Screenshot 2023-12-12 223203](https://github.com/reynoldputra/azure-cloud-computing/assets/100693456/0adb4179-65a1-4e6e-bb32-cec1905b19d7)

![Screenshot 2023-12-13 110543](https://github.com/reynoldputra/azure-cloud-computing/assets/100693456/fdefca82-888f-4222-88d4-fa00137bb71e)
Setelah itu, buka env dari resource group, buka azure fileshare (di tab settings). Daftarkan fileshare yang telah dibuat kedalam env resource group. (storage account key didapatkan pada storage account ditab security -> access keys)
![Screenshot 2023-12-13 110640](https://github.com/reynoldputra/azure-cloud-computing/assets/100693456/068039d4-0461-4b4d-8d4c-f949d67f2701)

Setelah itu, Di dalam container app mongodb, pilih tab application -> scale and replicas dan klik edit and deploy. Setelah itu klik mongo di container image. Pilih volume mounts seperti ss dibawah ini :

![Screenshot 2023-12-13 110816](https://github.com/reynoldputra/azure-cloud-computing/assets/100693456/98b4e637-7156-484a-8edd-17128041c427)

## Hasil Pengujian Enpoint

GET /orders

![image](https://github.com/reynoldputra/azure-cloud-computing/assets/112788819/c8ae4ad1-f85d-4f10-aa1a-d3f6c55945e4)

GET /orders/:id

![image](https://github.com/reynoldputra/azure-cloud-computing/assets/112788819/039b9e33-f1ef-4475-8922-50f4472592ef)

POST /orders

![image](https://github.com/reynoldputra/azure-cloud-computing/assets/112788819/8d5ba59e-6d09-4349-8b7b-c05037f374f9)

PUT /orders/:id

![image](https://github.com/reynoldputra/azure-cloud-computing/assets/112788819/edd7086c-8469-4162-b12e-c3660c3e0ae4)

DELETE /orders/:id

![image](https://github.com/reynoldputra/azure-cloud-computing/assets/112788819/136a7938-0390-40d7-809c-20f28b9d1de5)


## Load Testing

Berikut adalah spreadsheet rincian lengkap hasil percobaan endpoint menggunakan locust

![image](https://github.com/reynoldputra/azure-cloud-computing/assets/112788819/f73b02b7-e3a1-41a2-8c00-bed7e9e5d359)

spreadsheet dapat juga diakses melalui link berikut:
https://docs.google.com/spreadsheets/d/1xnf9qPY7j4fKis_jZZvuPYcSw2ECi9bnG723CFreYI0/edit#gid=50051680

### RPS
| Request | RPS | Fail | Fail % |
|---------|-----|------|--------|
| 18672   | 311 | 96   | 1.6    |
| 4269    | 150 | 0    | 0      |

### Peak Concurrent
| Spawn Rate | Peak Concurreny | Fail | Fail % |
|------------|-----------------|------|--------|
| 25         | 1475            | 0    | 0      |
| 50         | 3000            | 3    | 0      |
| 100        | 5700            | 0    | 0      |

## Pricing
![image](https://github.com/reynoldputra/azure-cloud-computing/assets/87769109/d388a2f8-1f18-440f-bfd0-c9aeba69884d)


## Kesimpulan
- Menggunakan Azure Container Apps mempermudah konfigurasi scalling
- Biaya lebih murah karna resouse menyesuaikan kebutuhan request
- Namun tidak dapat menerima request banyak secara tiba tiba (spike request) karena container memebutuhkan waktu untuk membuat replika
- Tetapi hal ini bisa dihindari dengan membuat spec container yang lebih besar dan jumlah concurrent pada scale rule yang rendah
- Azure Container Apps cocok digunakan untuk system dengan komputasi rendah, short running, dan jumlah requestnya terprediksi
