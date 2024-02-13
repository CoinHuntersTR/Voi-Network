<h1 align="center">Voi Network</h1>

> Adım adım yaptığınızda sorunsuz kurulum yaparsınız.

> Testnette kazanılan tüm tokenlerin mainnette ödül olarak dağıtılacak.

> Testnet başlayalı uzun süre oldu. Dileyen kurup bir yerinden katılabilir.

#

> ÖNEMLİ LİNKLER: [Voi Discord](https://discord.gg/voi-network-1055863853633785857) - [Whitepaper](https://afaf83a4-6c33-4e2a-a40c-9999410c0063.filesusr.com/ugd/7dc173_8e16834f2fbd4866a957d441f392d578.pdf)

## Donanım ve Güncelleme

### Sistem gereksinimleri:
#### Ubunutu 22.04
NODE TİPİ | CPU     | RAM      | SSD     |
| ------------- | ------------- | ------------- | -------- |
| Voi  | 4          | 8         | 80  |

4 CPU 8 RAM 80 SSD - Ubuntu 22.04
> Lokasyon olarak Amerika bölgesinden sunucu kiralayabilirseniz, daha fazla miktarda token kazanabilirsiniz.

### Güncellemeler:
```
sudo apt update && sudo apt-get upgrade -y
sudo systemctl start unattended-upgrades && sudo systemctl enable unattended-upgrades
```

### Kurulum

```
sudo apt install -y jq gnupg2 curl software-properties-common
curl -o - https://releases.algorand.com/key.pub | sudo tee /etc/apt/trusted.gpg.d/algorand.asc
```
> Çıktısına ENTER diyebiliriz.

```
sudo add-apt-repository "deb [arch=amd64] https://releases.algorand.com/deb/ stable main"
```
> Tekrar güncelleyelim ve nodeun otomatik başlamaması için durduralım:

```
sudo apt update && sudo apt install -y algorand && echo OK
```
```
sudo systemctl stop algorand && sudo systemctl disable algorand && echo OK
```
> goal setupı yapalım

```
echo -e "\nexport ALGORAND_DATA=/var/lib/algorand/" >> ~/.bashrc && source ~/.bashrc && echo OK
```
```
sudo adduser $(whoami) algorand && echo OK
```
> yapılandırma işlem:
```
sudo algocfg set -p DNSBootstrapID -v "<network>.voi.network" -d /var/lib/algorand/ &&\
sudo algocfg set -p EnableCatchupFromArchiveServers -v true -d /var/lib/algorand/ &&\
sudo chown algorand:algorand /var/lib/algorand/config.json &&\
sudo chmod g+w /var/lib/algorand/config.json &&\
echo OK
```
### Genesis
```
sudo curl -s -o /var/lib/algorand/genesis.json https://testnet-api.voi.nodly.io/genesis &&\
sudo chown algorand:algorand /var/lib/algorand/genesis.json &&\
echo OK
```

## Node Çalıştıralım

```console
# Algorandı Voi olarak yapılandıralım:
sudo cp /lib/systemd/system/algorand.service /etc/systemd/system/voi.service &&\
sudo sed -i 's/Algorand daemon/Voi daemon/g' /etc/systemd/system/voi.service &&\
echo OK

# ve nodeu çalıştralım:
sudo systemctl start voi && sudo systemctl enable voi && echo OK

# nodeu kontrrol edelim status ile:
goal node status

# ==> Genesis ID: voitest-v1
# ==> Genesis hash: IXnoWtviVVJW5LGivNFc0Dq14V3kqaXuK2u5OQrdVZo=
# Çıktının sonu bu şekilde olmalı (hash değişebilir)

# Hızlı sync olalım:
goal node catchup $(curl -s https://testnet-api.voi.nodly.io/v2/status|jq -r '.["last-catchpoint"]') &&\
echo OK
# Burada bir kaç dakika bekleyelim

# Yine status yapalım ama bu sefer loglarda Catchpoint göreceğiz:
goal node status

# Bu komutla kontrol ettiğimizde Sync Timeın sıfırlanmasını ve loglarda Catchpointin gitmesini bekleyelim.
goal node status -w 1000
# Yukarda ki şartlar gerçekleince CTRL + C
```

<h1 align="center">Ödül alabilmek için Telemtry yapalım</h1>

```console
# RuesTest yazan kısmı düzenleyiniz ve tırnakları <> kaldırın
sudo ALGORAND_DATA=/var/lib/algorand diagcfg telemetry name -n <RuesTest> - RuesCommunity

sudo ALGORAND_DATA=/var/lib/algorand diagcfg telemetry enable &&\
sudo systemctl restart voi
```

<h1 align="center">Cüzdan oluşturma işlemleri</h1>

```console
# Cüzdan oluşturalım:
goal wallet new voi
# Şifre belirledikten sonra Y diyip 24 kelimenizi alıp saklayın.

# Şimdi cüzdanımızı nodeumuza import edelim:
goal account import
# Şifre ve 24 kelimeyi girince bize bir Imported adres verecek bunu saklayalım cüzdan adresimiz.

# Şimdi bu kodları girelim ve bizden Imported adresimizi isteyecek.
# 1 Kerede kopyala yapıştır yapabilirsiniz bu kodu
echo -ne "\nEnter your voi address: " && read addr &&\
echo -ne "\nEnter duration in rounds [press ENTER to accept default (2M)]: " && read duration &&\
start=$(goal node status | grep "Last committed block:" | cut -d\  -f4) &&\
duration=${duration:-2000000} &&\
end=$((start + duration)) &&\
dilution=$(echo "sqrt($end - $start)" | bc) &&\
goal account addpartkey -a $addr --roundFirstValid $start --roundLastValid $end --keyDilution $dilution
# Imported adresinden sonra ki soruda ENTER diyip varsayılanı tercih edebiliriz.
# Import işleminin tamamlanmasını bekleyin ve Participation IDinizi saklayın.

# Aktifliğimize bakalım, burada çıktı !!OFFLİNE OLMALI!!
checkonline() {
  if [ "$addr" == "" ]; then echo -ne "\nEnter your voi address: " && read addr; else echo ""; fi
  goal account dump -a $addr | jq -r 'if (.onl == 1) then "You are online!" else "You are offline." end'
}
checkonline
```

> Bu aşamadan sonrasına devam etmek için discorddan token alın. `node-runners kanalı` => `/voi-testnet-faucet` şeklinde.

> Imported adresiniz ile [Explorerdan](https://voi.observer/explorer/home) kontrol edin token gelince devam edin.

<h1 align="center">Token Aldıktan Sonra</h1>

```console
# Tokenimizi aldıysak bu komutla !!Online olalım!!
getaddress() {
  if [ "$addr" == "" ]; then echo -ne "\nEnter your voi address: " && read addr; else echo ""; fi
}
getaddress &&\
goal account changeonlinestatus -a $addr -o=1 &&\
sleep 1 &&\
goal account dump -a $addr | jq -r 'if (.onl == 1) then "You are online!" else "You are offline." end'
```

![image](https://github.com/ruesandora/Voi/assets/101149671/6b030e34-9619-4191-a136-6312f94ba7cb)


<h1 align="center">Node kurduktan sonra yapılacaklar</h1>

> Node kurduktan `1-2 gün` sonra `#VoiScout-testnet` kanalında cüzdanınızın ilk 6 hanesini search edin.

> Zaten `online` iseniz doğru ama ek olarak burda gözüküyorsa prop execute etmişsiniz demektir. (iyi bir şey, node calısmıyor anlamına gelmez)

> [Buradan](https://cswenor.github.io/voi-proposer-data/health.html) kendimizi kontrol edelim (`Hour`)
