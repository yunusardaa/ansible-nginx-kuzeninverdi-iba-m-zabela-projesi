# Nginx Yüksek Erişilebilirlik (HA) Projesi - Ansible Otomasyonu

Bu proje, iki Ubuntu sunucusu üzerinde `Nginx` ve `Keepalived` kullanarak "Aktif-Pasif" yüksek erişilebilirlikli bir mimariyi **Ansible ile otomatik olarak** kurar.

Bu Playbook, `webservers` olarak tanımlanan sunuculara bağlanır ve aşağıdaki görevleri otomatik olarak gerçekleştirir:
* Nginx ve Keepalived kurulumu.
* IP yönlendirmenin (Sanal IP için) aktifleştirilmesi.
* Sunucunun rolüne (`master` veya `backup`) göre özel `index.html` dosyalarının kopyalanması.
* Role göre özel `keepalived.conf` (MASTER ve BACKUP) dosyalarının kopyalanması.
* Sadece MASTER sunucuya otomatik yedekleme script'i ve `cron` görevi eklenmesi.

## Mimari

* **Kontrol Düğümü (Control Node):** Ansible komutlarının çalıştırıldığı sunucu.
* **Web Sunucuları (Hedefler):**
    * `MASTER` Rolü: Nginx'i çalıştıran ve Sanal IP'yi tutan birincil sunucu.
    * `BACKUP` Rolü: MASTER çöktüğünde Sanal IP'yi devralan ikincil sunucu.

## Kurulum ve Çalıştırma

Bu otomasyonu çalıştırmak için 3 sunucuya ihtiyacınız vardır: 1 Kontrol Düğümü (Patron) ve 2 Hedef Sunucu (Master, Backup).

### 1. Hazırlık: Kontrol Düğümü (Patron Sunucu)

1.  Ansible'ı kurun:
    ```bash
    sudo apt update
    sudo apt install ansible -y
    ```
2.  Hedef sunuculara (Master ve Backup) şifresiz SSH erişimi için anahtarınızı kopyalayın:
    ```bash
    ssh-keygen
    ssh-copy-id kullanici@MASTER_SUNUCU_IP
    ssh-copy-id kullanici@BACKUP_SUNUCU_IP
    ```

### 2. Envanter (Inventory) Hazırlığı

1.  Bu depodaki `hosts.example` dosyasını `/etc/ansible/hosts` olarak kopyalayın.
2.  `/etc/ansible/hosts` dosyasını açın ve IP adreslerini kendi sunucu IP'lerinizle değiştirin.

### 3. Playbook'u Çalıştırma

1.  Bu depodaki `ansible.cfg` ve `deploy.yml` dosyalarını Kontrol Düğümünüzün ev dizinine (`~/`) indirin.
2.  Proje dizinindeyken (`cd ~`) Playbook'u çalıştırın:
    ```bash
    ansible-playbook deploy.yml -K
    ```
3.  Komut sizden `BECOME password:` (yani `sudo` şifresi) isteyecektir. Sunucularınızın `sudo` şifresini girin.

Playbook bittiğinde (`failed=0` olarak), `http://[virtual_ip]` adresinden (Sanal IP) sitenizi kontrol edebilirsiniz.
