
# Konfigurasi Firewall Rules

## Konfigurasi Firewall *Allow HTTP*
1. Dari navigasi menu, pilih VPC Network > Firewall
![menu firewall](images/1.png)

2. Pada halaman Firewall, klik tombol **CREATE FIREWALL RULE**
![create firewall rule](images/2.png)

3. Isi form sesuai dengan requirement:
    >**Name**: dicoding-allow-http\
    >**Targets**: Specified target tags\
    >**Target tags**: dicoding-http-server\
    >**Source filter**: IPv4 ranges\
    >**Source IPv4 ranges**: 0.0.0.0/0\
    >**Protocols and ports**: 
    >- Specified protocols and ports 
    >    -  TCP : 80
    >
    >**lainnya**: default

    ![create firewall rule](images/3.png)
    ![create firewall rule](images/4.png)

4. Jika sudah selesai, tekan tombol **CREATE**
    #### Proses diatas, juga dapat dilakukan via console, dengan script dibawah
    ```c
    gcloud compute --project=gcp-dicoding-agungr firewall-rules create dicoding-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=dicoding-http-server
    ```

## Konfigurasi Firewall *Allow HTTP Check*
1. Pada halaman Firewall, klik tombol **CREATE FIREWALL RULE**
![create firewall rule](images/2.png)

2. Isi form sesuai dengan requirement:
    >**Name**: dicoding-allow-httpcheck\
    >**Targets**: Specified target tags\
    >**Target tags**: dicoding-http-server\
    >**Source filter**: IPv4 ranges\
    >**Source IPv4 ranges**: 130.211.0.0/22, 35.191.0.0/16\
    >**Protocols and ports**: 
    >- Specified protocols and ports, centang **TCP** tanpa mengisi port
    >**lainnya**: default
        
    ![create firewall rule](images/5.png)
    ![create firewall rule](images/6.png)

4. Jika sudah selesai, tekan tombol **CREATE**
    #### Proses diatas, juga dapat dilakukan via console, dengan script dibawah
    ```c
    gcloud compute --project=gcp-dicoding-agungr firewall-rules create dicoding-allow-httpcheck --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp --source-ranges=130.211.0.0/22,35.191.0.0/16 --target-tags=dicoding-http-server
    ```

## Konfigurasi Instance Templates dan Instance Groups

>Untuk membuat load balancer, kita perlu menggunakan Managed Instance Group. Untuk membuatnya, kita memerlukan Instance Template untuk Autoscaling.

1. Masuk Navigation menu pilih **Compute Engine** > **Instance Template** 
![menu instance template](images/7.png)

2. Klik **CREATE INSTANCE TEMPLATE**
![create instance template](images/8.png)

3. Beri nama Instance
![give name of instance template](images/9.png)

4. Karena hanya menampilkan web sederhana, saya sesuaikan **Machine Configuration** dengan spec yang lebih rendah
![machine configuration of instance template](images/10.png)

5. Buka panah pada **Networking, Disks, Security, Management, Sole-Tenancy**

6. Pada bagian **Networking**, isi network tags dengan tag yang sudah dibuat sebelumnya
![networking of instance template](images/11.png)

7. Pada bagian **Management** isi startup script dengan script berikut untuk melakukan instalasi web server apache
    ```c
    sudo apt-get update
    sudo apt-get install apache2 -y
    echo '<!doctype html><html><body><h1>Hello from Jakarta!<h1></body></html>' | sudo tee /var/www/html/index.html
    ```
    ![automation script in instance template](images/12.png)

8. Jika sudah, tekan tombol **CREATE**

9. Kemudian buat 1 Instance Template lagi untuk region europe-west1. centang template **asia-southeast2-template**, kemudian tekan **COPY**
![copy instance template](images/13.png)

10. Ganti nama template *europe-west1-template*
![copy instance template](images/14.png)

11. Buka panah pada **Networking, Disks, Security, Management, Sole-Tenancy**. Pada bagian **Management** rubah script html
    ```c
    sudo apt-get update
    sudo apt-get install apache2 -y
    echo '<!doctype html><html><body><h1>Hello from Europe!<h1></body></html>' | sudo tee /var/www/html/index.html
    ```
    ![automation script in instance template](images/15.png)

12. Jika sudah tekan **CREATE**

13. Maka saat ini sudah ada 2 template Instance yang sudah dibuat
![instance template list](images/16.png)

Selanjutnya kita akan buat instance group yang bersumber dari instance template yang sudah dibuat tadi.

1. Buka halaman **Instance Group**
![open instance group](images/17.png)

2. Tekan tombol **CREATE INSTANCE GROUP**


3. Isi pada form seperti berikut:
    >**Name** : asia-southeast2-group\
    >**instance template** : asia-southeast2-template\
    >**location** : multizone, region: asia-southeast2\
    >**minimum instance** : 1\
    >**maximum instance** : 5\
    >**Metric CPU Utilization** : 80%

    ![create instance group](images/18.png)
    ![create instance group](images/19.png)
    ![create instance group](images/20.png)

4. Setelah itu tekan **CREATE**

5. Maka pada halaman instance grup akan terbentuk instance group baru
![instance group list](images/21.png)

6. Masuk ke menu **VM Instance**, maka akan terbentuk instance group baru dengan nama *asia-southeast2-group-xxxx*
![instance vm list](images/22.png)

7. Browse External IP VM tersebut untuk memastikan bahwa web server sudah berjalan
![instance vm list](images/23.png)

8. Jika tampilan seperti ini, maka web server sudah berjalan sesuai template yang sudah dibuat
![instance vm list](images/24.png)

9. Selanjutnya, kita akan buat 1 group instance untuk region europe.
![create instance group](images/25.png)
![create instance group](images/26.png)
![create instance group](images/27.png)

10. Saat ini sudah ada 2 instance group
![create instance group](images/28.png)

11. Dan kita bisa lakukan pengecekan kembali di menu **VM INSTANCE** untuk memastikan web server pada europe sudah bisa diakses
![wm europe](images/29.png)

Saat ini kita sudah berhasil membuat 2 instance group yang akan kita gunakan sebagai Backend HTTP Load balancer


## Konfigurasi HTTP Load Balancer

1. Buka halaman load balancing melalui **Navigation Menu** -> **Network services** -> **Load balancing**.
![create load balance](images/30.png)

2. Klik **Create Load Balancer** untuk membuat.
![create load balance](images/31.png)

3. Pilih **Start configuration** pada HTTP(S) Load Balancing. 
![start config](images/32.png)

4. Biarkan opsi menggunakan default, dan klik **Continue**
![start config](images/33.png)

5. Beri nama Load Balancer, kemudian pada bagian **Backend configuration**, klik kolom **Backend services & backend buckets**, lalu pilih **Create a Backend Service**
![start config](images/34.png)

6. Beri nama backend service, dan pilih backend type Instance group dengan protocol http
![start config](images/35.png)

7. Pada kolom backend, isi sebagai berikut, setelah itu klik **DONE**:\
 **Instance group** : **asia-southeast2-group** \
 **Port numbers** : 80 \
 **Balancing mode** : Rate \
 **Maximum RPS** : 50 \
 **Lainnya** : default
![start config](images/36.png)

8. Tambahkan Backend baru seperti berikut:\
 **Instance group** : europe-west1-group \
 **Port numbers** : 80 \
 **lainnya** : default
![start config](images/37.png)

9. Pada kolom **Health check**, tekan **Create a Health Check**
![start config](images/38.png)

10. Beri nama, ubah protocol menjadi **HTTP**, dan klik **Save**
![start config](images/39.png)

11. Kemudian klik **Create** untuk menyimpan, dan akan muncul tabel list **Backend services**
![start config](images/40.png)

12. Pilih **Frontend configuration**. Pilih IPv4 pada IP version, pilih Ephemeral pada IP address, dan pilih 80 pada Port. Lalu, klik Done.
![start config](images/41.png)
![start config](images/42.png)

13. Tambahkan lagi Frontend configuration dengan klik Add Frontend IP and Port. Pilih IPv6 pada IP version, pilih Ephemeral pada IP address, dan pilih 80 pada Port. Lalu, klik Done.
![start config](images/43.png)
![start config](images/44.png)

14. Maka akan muncul 2 list Frontend Configuration
![start config](images/45.png)

15. Review and finalize untuk mengulas kembali konfigurasi dari load balancer yang akan kita buat. Jika sudah benar, klik tombol Create
![start config](images/46.png)
![start config](images/47.png)

16. Beberapa menit kemudian, Load Balancer sudah siap digunakan
![start config](images/48.png)

17. Copy address pada kolom Address di tab **FrontEnds**. dan buka pada browser 
![start config](images/49.png)

18. Jika sudah benar, akan muncul web yang sudah dibuat sebelumnya
![start config](images/50.png)


## Stress Test terhadap Load Balancer

Langkah selanjutnya, kita perlu melakukan stress test untuk menguji load balancer yang sudah kita buat berjalan dengan baik atau tidak

1. Buka Cloud Shell dan jalankan perintah berikut untuk menginstall siege. Siege adalah tools yang umum digunakan untuk melakukan stress test.
    ```c
    sudo apt-get install siege -y
    ```
    ![install siege](images/51.png)

2. Jalankan perintah berikut untuk memulai stress test
    ```c
    siege -c 250 http://load_balancer_ip
    ```
    ![stress test](images/52.png)

3. Ketika stress test dijalankan, konsol akan tampak seperti berikut atau blank seperti diatas
    >** SIEGE 4.0.4\
    >** Preparing 250 concurrent users for battle.\
    >The server is now under siege...
    

4. Buka halaman **Load balancing**, lalu klik pada nama load balancer Anda. 
![load balance](images/53.png)

5. Buka tab **Monitoring**, Anda akan melihat alur dari traffic yang dikirim dari Siege. Karena kita telah mengatur backend di asia-southeast2 hanya bisa menerima 50 RPS, traffic pun akan dialihkan ke europe-west1 meskipun VM tempat kita mengakses berada di Asia.
![monitoring lb](images/54.png)

6. Jika kita buka browser lagi, maka kita akan diarahkan secara otomatis ke server Asia maupun ke Eropa sesuai dengan Load Balancer yang sudah kita buat sebelumnya
![monitoring lb](images/55.png)
![monitoring lb](images/56.png)


# Sekian Terima Kasih
