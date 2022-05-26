
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
