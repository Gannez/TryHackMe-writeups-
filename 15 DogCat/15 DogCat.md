<style>
@page {
  size: A4;
  margin: 25mm;
}

html, body {
  
  color: #ffffff;
  font-family: "Times New Roman", serif;
  font-size: 12pt;
  line-height: 1.5;
}

body {
  max-width: 160mm;
  margin: 0 auto;
  padding: 0;
}

.cover {
  min-height: 247mm;
  display: flex;
  flex-direction: column;
  text-align: center;
  page-break-after: always;
  break-after: page;
}

.logo {
  text-align: center;
  margin-top: 12mm;
}

/* Зөвхөн логонд үйлчлэх хэсэг */
.logo img {
  width: 200px;
  height: auto;
  object-fit: contain;
}

/* Тайлан болон бусад хэсэгт орж буй зургийг голлох болон хэмжээг тохируулах */
img:not(.logo img) {
  display: block;
  margin: 15px auto;
  max-width: 100%;
  height: auto;
}

.school {
  font-size: 15pt;
  font-weight: bold;
  margin-top: 4mm;
}

.department {
  font-size: 14pt;
  font-weight: bold;
  margin-top: 2mm;
}

.report-title {
  font-size: 24pt;
  font-weight: bold;
  line-height: 1.35;
  margin-top: 18mm;
}

.course {
  font-size: 15pt;
  margin-top: 8mm;
}

.details {
  text-align: left;
  margin: 25mm 8mm 0 8mm;
  line-height: 2;
}

.city {
  margin-top: auto;
  padding-bottom: 2mm;
  line-height: 1.8;
}

h1, h2, h3 {
  color: #000000;
}

h1 {
  font-size: 16pt;
}

p {
  text-align: justify;
}
</style>

<div class="cover">

<div class="school">
  ШИНЖЛЭХ УХААН ТЕХНОЛОГИЙН ИХ СУРГУУЛЬ
</div>

<div class="department">
  Мэдээлэл холбооны технологийн сургууль
</div>
<div class="logo">
  <img src="logo.jpg" alt="ШУТИС-ийн лого">
</div>

<div class="report-title">
  БИЕ ДААЛТЫН АЖЛЫН<br>
  ТАЙЛАН
</div>

<div class="course">
  Веб системийн аюулгүй байдал /F.NSA342/<br>
  (2026-2027 оны хичээлийн жилийн Намар)
</div>

<div class="details">

**Машины нэр:**  Dogcat (MEDIUM)   
**Хичээл заасан багш:** Ц.Энхтөр /FSN07/

**Лабораторийн цаг:** 2/2

**Бие даалтын ажил гүйцэтгэсэн оюутан:** Г.Ганбаяр B231870023

</div>

<div class="city">
  Улаанбаатар хот<br>
  2026 он
</div>

</div>




# TRYHACKME – DOGCAT

## УДИРТГАЛ

Энэхүү даалгаврын хүрээнд TryHackMe платформын **Dogcat** виртуал машины аюулгүй байдлын түвшинг шалгав. Вебийн параметрээр файл унших (LFI) сул талыг илрүүлж, Apache access log-ийг ашиглан PHP код ажиллуулж `www-data` shell авсан. Дараа нь root эрхээр ажилладаг backup script-ийн бичих эрхийн алдааг ашиглан root shell авсан.

## ВЕБ ХУУДСЫГ ШАЛГАЖ DIRECTORY TRAVERSAL / LFI-Г СЭЖИГЛЭХ

`curl` ашиглан веб хуудасны HTML, `view=dog` зэрэг параметрүүдийг шалгасан. PHP backend нь хэрэглэгчийн оруулсан параметрээр файл сонгон унших шинжтэй байв. `../` болон URL encoding-тай хувилбаруудыг туршиж файлын замыг дамжуулах хамгаалалтыг тойрон гарах боломжийг шалгасан. Энэ нь **Local File Inclusion (LFI) / Path Traversal** төрлийн эмзэг байдалтай нийцэж байна.

## APACHE ACCESS LOG-ООР PHP КОДЫН ГҮЙЦЭТГЭЛ ШАЛГАХ

Дараагийн зургууд Apache access log руу хандах, HTTP request-ийн `User-Agent` толгойг ашиглан тэмдэгт оруулах, улмаар PHP код ажилласныг `RCE_CONFIRMED_123`, `PHP_EXEC_CONFIRMED` зэрэг хариугаар шалгасан ажиллагааг харуулна. Эмзэг файлын уншилтыг логийн кодтой холбосноор **log poisoning → remote code execution** вектор үүссэн.

## REVERSE SHELL АВАХ

Kali талд `nc -lvnp 4444` сонсох горим ажиллуулж, зорилтот хостоос эргэж холбогдсон. `whoami`, `id`-ийн хариунд `www-data`, `uid=33(www-data)` гарсан. Түүнчлэн `/var/www/html` орчинд ажиллаж байгааг баталсан.

## КОНТЕЙНЕРИЙН БОЛОН НӨӨЦЛӨЛТИЙН ТОХИРГООГ ШАЛГАХ

`/root`, `/var/www`, `/opt/backups` зэрэг хавтсыг шалгаж flag-уудыг хайсан. `/opt/backups/backup.sh` нь `tar` ашиглан `/root/container`-ийг backup archive болгодог скрипт байгааг илрүүлсэн. Нөөцлөх процесс илүү өндөр эрхээр ажиллах үед тухайн скриптийг өөрчилж болох сул тал эрх нэмэгдүүлэх вектор болсон.

## BACKUP SCRIPT АШИГЛАН ROOT REVERSE SHELL АВАХ

Зургуудад `backup.sh` руу холболт үүсгэх shell мөр нэмсэн, Kali дээр `5555` порт сонссон, дараа нь буцаж ирсэн shell-д `whoami = root`, `id = uid=0(root)` гарсан байна. `hostname = dogcat`. Дараа нь `/root/flag4.txt`-ийг уншсан. Иймээс **Dogcat машины root shell болон эцсийн flag баталгаажсан**.

## ДЭЛГЭЦИЙН ЗУРГИЙН НОТОЛГОО

<img src="image1.png" alt="Dogcat ажиллагааны нотолгоо 01">

<img src="image2.png" alt="Dogcat ажиллагааны нотолгоо 02">

<img src="image3.png" alt="Dogcat ажиллагааны нотолгоо 03">

<img src="image4.png" alt="Dogcat ажиллагааны нотолгоо 04">

<img src="image5.png" alt="Dogcat ажиллагааны нотолгоо 05">

<img src="image6.png" alt="Dogcat ажиллагааны нотолгоо 06">

<img src="image7.png" alt="Dogcat ажиллагааны нотолгоо 07">

<img src="image8.png" alt="Dogcat ажиллагааны нотолгоо 08">

<img src="image9.png" alt="Dogcat ажиллагааны нотолгоо 09">

<img src="image10.png" alt="Dogcat ажиллагааны нотолгоо 10">

<img src="image11.png" alt="Dogcat ажиллагааны нотолгоо 11">

<img src="image12.png" alt="Dogcat ажиллагааны нотолгоо 12">

<img src="image13.png" alt="Dogcat ажиллагааны нотолгоо 13">

<img src="image14.png" alt="Dogcat ажиллагааны нотолгоо 14">

<img src="image15.png" alt="Dogcat ажиллагааны нотолгоо 15">

<img src="image16.png" alt="Dogcat ажиллагааны нотолгоо 16">

<img src="image17.png" alt="Dogcat ажиллагааны нотолгоо 17">

<img src="image18.png" alt="Dogcat ажиллагааны нотолгоо 18">

<img src="image19.png" alt="Dogcat ажиллагааны нотолгоо 19">

<img src="image20.png" alt="Dogcat ажиллагааны нотолгоо 20">

<img src="image21.png" alt="Dogcat ажиллагааны нотолгоо 21">

<img src="image22.png" alt="Dogcat ажиллагааны нотолгоо 22">

<img src="image23.png" alt="Dogcat ажиллагааны нотолгоо 23">

<img src="image24.png" alt="Dogcat ажиллагааны нотолгоо 24">

<img src="image25.png" alt="Dogcat ажиллагааны нотолгоо 25">

<img src="image26.png" alt="Dogcat ажиллагааны нотолгоо 26">

<img src="image27.png" alt="Dogcat ажиллагааны нотолгоо 27">

<img src="image28.png" alt="Dogcat ажиллагааны нотолгоо 28">

<img src="image29.png" alt="Dogcat ажиллагааны нотолгоо 29">

## ХАМГААЛАЛТЫН ЗӨВЛӨМЖ

**Хамгаалалт:** `include`/файл замын параметрийг зөвшөөрөгдсөн утгын жагсаалтаар хязгаарлах; canonical path validation хэрэглэх; серверийн логийг PHP include-д хэзээ ч ашиглахгүй байх; log directory-д executable эрх өгөхгүй байх; root-аар ажилладаг backup script-үүдийн эзэмшил, permission-ийг зөв тохируулах; container mount, cron болон backup integrity-г хянах; аномаль HTTP request, reverse shell egress-ийг илрүүлэх.

## DOGCAT — ЭЦСИЙН ДҮГНЭЛТ

Dogcat дээр файлд хандах алдаа болон лог боловсруулахтай холбоотой сул тал нийлж `www-data` эрхтэй shell-д хүргэсэн. Дараа нь root эрхээр ажилладаг backup script-ийн бичих эрхийн алдааг ашигласан нь root түвшинд хүрэх боломж бүрдүүлсэн. `whoami`, `id` болон root flag-ийн гаралт root shell авсныг баталж байна.
