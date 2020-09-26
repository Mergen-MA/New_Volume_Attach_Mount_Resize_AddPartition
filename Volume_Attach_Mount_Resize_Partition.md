AWS Adding Volume and Attach on Instance
 
Ephemeral (Instance) : Temporary -RAM gibi ama tam olarak RAM degil.
Daha EBS ile ilgilenecegiz.  SSD ve HDD olarak 2’ye ayrılır. 
Sanal makine hypvisor üzerinde çalısıyor. Instance store direkt olarak hypvisore baglı olarak çalışır. Instance storage sadece bir instance’a bağlı oalrak çalışır. Bu da en yakın olanıdır. 
 
S3 object based bir storage. EBS block based’tir. PC deki hardisk gibidir. 

 
-->	Default olarak General-Purpose SSD(GP2).
-->	Thoroughput: Saniyede ne kadar yazıldıgını gösterir. Büyük log dosyaları, big data datware houselar akla gelmeli. IOPS’a 	göre daha iyi performans. 
-->	IOPS: Input-output per second: Okuma yazma hızı. Okuma yazma isleminin cok yapıldıgı yerlerde kullanılır.
-->	Sizin için IOPS önemliyse SSD’e yönel. Throughput önemliyse Harddisklere yönel.
-->	SSD: IOPS, HDD: throughput

What will we do in this Hands-on?

Managing EBS Volumes on Console and Terminal:

-->	Attaching
-->	Detaching
-->	Mounting
-->	Partition
-->	Resizing (single partition)-Volume değiştirme

1.	Instance oluştur (IAM user best practice-N.Virginia).
Df df 
-->	Root device type:ebs şeklinde gözükür. Sanal makinenin altında.
-->	Configure instance baslıgı altında subnet us-east-1a seç. 
-->	Add storage’ta volume’ları incele. Volume type altından secebilrisin. General purpose SSD (gp2)
-->	Security group SSH’lı olsun yeter. HTTP ile beraber de olabilir. 

2.	Terminali aç. Instance’a bağlan. (8gb’lık volume attach şeklinde geldi)

3.	lsblk komutu: volume’lara bakıyoruz. EC2’da ki volume’a baktık. 


4.	AWS Console --> Instance page --> Elastic Block Store-->Volumes’a tıkla
Create Volume tıkla
	-Açılan pencerede:
	   -Volume type: General Purpose-gp2
	   -size:ekleyeceğin değeri gir (1....16284GİB)
	   -availability zone: Instance ile volume aynı zone’da olmak zorunda.Create volume tıkla.
5. Oluşturduğumuz volume’u root volume’a (8gb) attach edecegiz. (30 Gb’ı geçme eklerken)
      -Yeni volume’ı seç. 
      	- Actions
	     -Instance içinde root volume’ı seç.
	     -Device : hangi isimle nereye oluşturacağını yazar.
	     -Attach’e tıkla. Bitti.
Terminale geç.

6. lsblk komutu calıştır. 
	Attach ettiğin volume’u gör. Linux bu volume’ı gördü. 

7. sudo file -s /dev/xvdf  --> formatına bakıyoruz. Herhangi bir formatta göstermiyor. Data şeklinde gözüküyor. Attach etmek için formtlamak gerekiyor. Bir disk ilk oluştrulduğunda formatı olmaz.

8. sudo mkfs -t ext4 /dev/xvdf --> Diski formatladık şuan ext4 formatına dönüştürdük.

9. sudo file -s /dev/xvdf --> tekrar formatına bakıyoruz. Bunu da bir klasörün altına eklememiz gerekiyor attach edebilmek için. Root’un altına klasör oluşturacağız.

10. sudo mkdir /mnt/2nd-vol/ --> directory oluştu. Bundan sonra mount islemini yapacağız. Önce ls yap directory’e bak.

11. cd --> home klasörüne gel.

12. sudo mount /dev/xvdf /mnt/2nd-vol (/dev/xvdf altındaki volume’u 2nd-vol’a ile ilişkinlendir- bu klasörün altına aldık.
	-Attach edip, formatlayıp, mount ettiğim volume’un içine girebilirim artık. 
13. lsblk

14. df --> elimizdeki dosyaları gösterir. Ne kadar kullanıldı, nereye bağlı oldgunu gösterir.

15. df -h --> daha okunaklı hale getirir.
   - Instance’ı volume attach ettik. Formatladık, mount ettik, klasör üzerinden de işletim sistemine eklemiş olduk. Lsblk ve df -h komutlarıyla görmüş olduk.

16.cd /mnt/2nd-vol -->  volume içine girdik.En baştan beri yapmak istediğim şeyi yaptım.
	ls--> lost+found şeklinde bir sey gözükür. 

17. sudo touch mavolume.txt--> yeni dosya oluşturdum. Bilgilerimiz orda duruyor mu, bilgilerimiz kayboluyor mu kaybolmuyor mu onu görelim.

18. ls --> dosya gözüküyor. 

19. Oluşturduğumuz volume yetmedi diyelim. Şimdi volume’ı modify edeceğiz. 
20. AWS Console--> Volumes--> 2GB lık volume’u seç. 
      -Actions
	-size: 4 GB yap
	-Modify’ı tıkla. 

21. Sağ üstten refresh yap.  

22. lsblk --> 4GB olduğunu gördük.

23. df -h -->  Dosya sistemi kısmından baktık su an. Burda 2.volume hala 2GB gözüküyor. Linux farkında aslında ama gerekli işlemleri yapılmamış. Dosyalama sistemiyle ilgili.

24. sudo resize2fs /dev/xvdf --> extend yaptık. 

25. df -h --> işlemin yapıldığını görmüş olduk. 
	2GB’lık harici bir volume oluşturduk. Attach ettik. Formatladık. Daha sonra management console’dan resize ettik 2’den 4’e çıkardık. 

26. cd /mnt/2nd-vol 

27. ls
Sudo 
28. diyelim ki EC2 instance stop oldu. Tekrar çalıstırmamız lazım.
	-sudo reboot now
29. Tekrar bağlan. Aynı IP adresi gecerli. Yukarı ok tuşuna bas komut görünecektir. 
	Connection refused derse biraz bekleyip tekrar bağlan. Tam reboot olmamıştır.

30. lsblk
31. df -h

32. Mount point’ler gözükmez. Sistem reboot edildiği zaman yaptığımız işlemler kaybolur.

33. sudo file -s /dev/xvdf --> file komutuyla dosy formatına bakıyoruz. Ext4 formatında volume var. Tekrar mount edeceğiz. 

34. sudo mount /dev/xvdf /mnt/2nd-vol

35. lsblk sonra df -h --> reboot tan sonra ilişkilendirme gitti. Reboot’tan sonra yaptığımız tek işlem mount yapmak. 

Partition İşlemine başlıyoruz.

1. Console’dan Create Volume (3.volume oluşturuyoruz)
	Size :2GB
	Availability zone:us-east-1a
2. Attach edeceğiz. Volume’u seç. 
	-Actions--> 8gb’lık root volume attach yap. 
3. lsblk – mount point yoktur. 
4. df -h
	Partition işlemini yapacağız. Windowstaki c, d mantığı.

5. sudo fdisk -l --> diskleri partition ları görürüz. Yeni eklediğimiz volume’ların partition’ı yok. 

6. sudo fdisk /dev/xvdg

7. m +Enter --> yardım menüsüne gir.
8. n + Enter--> add a partition. P ve e secenkleri çıkar.
9. p + Enter--> partition işlemi başlar. 
10. 1 + Enter --> hiç birşeye basmadan enterlarsan 1 verir. (1’den 4’e kadar verilebilir)
11. First sector (2048-4194303, default 2048) Enter layınca default değerden başlar.
12. Last Sector, +sector or +size{K,M,G,T,P} (2048-419303, default 4194303): 2100000
       1. partition oluşturuldu. Şimdi aynı işlemler 2.partition için yapılacak.
13. n + Enter--> add a partition. P ve e secenkleri çıkar.
14. p + Enter--> partition işlemi başlar. 
15.Partition number (2-4, default 2): Enter
16. First Sector (2100001-4194303, default 2101248): Enter
17.Last sector : default olması için enter.
        2.partition da ouşturuldu.
18. Command (m for help) : w --> Yaptığımız işlemler kaydedildi.
19.lsblk -->partitionları görüyoruz. 
	sudo

20. sudo  mkfs -t ext4 /dev/xvdg1
       1.partition formatlandı. İlgili işlemlari yapmak için. 
21. sudo  mkfs -t ext4 /dev/xvdg2sud
        2.partition formatlandı. İlgili işlemlari yapmak için. 

22.sudo mkdir /mnt/3rd-vol-part1 --> klasör açıyoruz buraya mount etmek için

23. sudo mkdir /mnt/3rd-vol-part2 --> klasör açıyoruz buraya mount etmek için
24.sudo mount /dev/xvdg1 /mnt/3rd-vol-part1 --> Bu klasöre mount ettik.
25. sudo mount /dev/xvdg2 /mnt/3rd-vol-part2 
26.lsblk
 

27. df -h 

28. cd /mnt/3rd-vol-part2
29.ls

30. sudo touch mavolume1.txt
31. ls

32. cd 
33. En son volume’u actions bölümünden modify ediyoruz. 
	Size: 3GB yap
	Modify
	refresh
34. lsblk--> Sistem görüyor. Partition’lar hala aynı büyüklükte.
35.df -h
36.sudo growpart /dev/xvdg 2
37.sudo resize2fs /dev/xvdg2
38. lsblk-->partititonların büyüdüğünü gör. 
39. cd /mnt/3rd-vol-part2
40.ls--> daha once olusturdugumuz dosyaya baktık.

41. sudo reboot now-->instance reboot ediliyor. 
42. yukarı tusuna bas.komutu goreceksin. Baglan. 
	Mount islemleri yine unmount oldu. Partitionlar duruyor. 

43. cd /etc/ --> bu klasor icinde auto mount yapmak icin bir dosyanın icinde degisiklik yapmamaız gerekiyor.

44. cd fstab
45.cat fstab --> bu dosyayı modifiye edececgiz. Önemli vir dosya olduğu için bunu kopyalayacagız once.
46. sudo cp fstab fstab.bak--> backup aldık.

47. sudo nano fstab--> modifiye islemine baslıyoruz.
48. cd /etc ile etc klasorune indik. oradaki fstab dosyasini nano ile acip, ekledigimiz volumelarin 
otomatik sekilde mount olmasi icin asagidaki kaydi bu klasore ekliyoruz.

/dev/xvdf      /mnt/2nd-vol            ext4   defaults, nofail 0  0
/dev/xvdg1     /mnt/3rd_volume-part1   ext4   defaults, nofail 0  0
/dev/xvdg2     /mnt/3rd_volume-part2   ext4   defaults, nofail 0  0

reboot ettikten sonra, instance a yeniden baglandik ve **lsblk** ile volume bilgilerini aliyoruz. otomatik olarak mount oldugunu gorduk.







