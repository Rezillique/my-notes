---
title: 2. Blogu sitede yayınlama
date: 2024-12-28
draft: false
tags:
  - nginx
  - blog
---


Hostinger gibi host şirketlerine para vermemek için bu işi elimdeki raspberry ile yapmaya karar verdim. Tabi birtakım problemlerim var öncelikle githuba pushladığım siteyi raspinin çekmesi lazım bu işin kolay kısmı gibi. Aslında yapılanları aşama olarak çizmem gerekirse aşağıdaki gibi olması lazım:

![[Screenshot 2024-12-28 051239.png]]

Çok kabaca bu şekilde olması lazım.  Bu blog serisini https://www.youtube.com/watch?v=dnE7c0ELEH8 linkteki video sayesinde yapmaya karar verdim. İşin ortasından anlatmaya başlamış olmama rağmen Github ve öncesi hazır (yani en azından öyle düşnüyorum) ve bu kısımları sonraki yazıda anlatmak derdindeyim. 

Github kısmı hazır. Raspi kısmı da hazır. Görselleri ekrana koymayacağım ama ilk önce SD kartı raspberry pi imager hazırlıyorum. Raspi 4 8Gb modelini kullandığım için 64-bit işlemcili raspbian işletim sistemini yüklüyorum. Bundan önceki projelerde de aynı işletim sistemini kullandım çünkü ubuntu, kali, mint vb. diğer distroların raspi için uyumlu versiyonları olsa bile hem kullanım kolaylığı hem de hafiflik açısından resmi raspbian kadar iş yapmadığı kanaatindeyim.  Headless start için SSH (Secure Shell) ile bağlanıp -ki Github ile bu konuda çok uğraştım- VNC özelliğini aktive edip desktop enviromentı ekranıma alarak bir huzura erme sürecine girdim. Her zaman olduğu gibi uğraşılan iş linuxtaysa güncelleme yapmak şart bu yüzden

"sudo su -"   komutunu yazarak terminali (Win kullanıcıları için CMD veya Powershell) super user (yönetici) izinleri verip 

"apt-get update"  güncellemeleri alıyorum sonrasında nginx kurulumu için

"apt-get install nginx" ile nginx indirip kurulumu tamamlıyorum.


Neden nginx? Raspi gibi küçük cihazlar için düşük hafıza ve yüksek performan önemli özellikler. Nginx bu özelliklere sahip olan web sitesi hostlamamı sağlayacak olan bir open source yazılım.
Başka yazılımlar mevcut ama istediğim özellikleri sağlamıyor veya kurulumu bu kadar rahat değil.

nginx kurulumu tamamlandıktan sonra bunu doğrulamak için 

"curl localhost"  yazarak bunu denetlemem lazım. 


![[Screenshot 2024-12-28 052951.png]]

Görselde görüldüğü gibi tarayıcıyı açıp localhost yazarak arama çubuğunu nginx kurulumunun başarılı olduğunu görüyorum. Laptopumda aynı ağa bağlı olduğu için raspi ip adresini arama çubuğuna yazarak aynı ekranı görüyorum.

![[Screenshot 2024-12-28 053318.png]]


Şimdi sıra githubta bulunan siteyi nginx'e aktarmak var.

Hiiiiiiç şaşırtmayacak şekilde saatlerimi ssh key ayarlamakla geçirdim. Ssh keyleri için bir parola oluşturmamış olmama rağmen ssh-agent denilen bir arka plan programa ihtiyaç duyuyorsun. Windowsta bu programı kendin çalıştırman gerekiyor hatta otomatik olarak başlatmak en iyisi linuxta zaten çalışıyor otomatik olarak. Linuxta bu işlemin otomatik olduğunu bilmeme rağmen yanlış anlamış olacağım ki oluşturduğun ssh keyi kendin eklemen lazım. Otomatik olarak demek ben bir key oluşturduğumda bunun otomatik eklenmesi demek değil bunu saatler sonra zor yoldan anlamış oldum. Bu arada ssh-key oluşturma komutu da 

```shell
ssh-keygen -t ed25519 -C "your_email@example.com"
```

veya 

```shell
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

rsa ve ed25519 şifreleme algoritmaları ki çok ayrı bir konu onlar belki daha sonra yazarım. rsa olanı kullanıyoruz.  Şimdi key oluşturuldu eklendi github tarafında authentication doğrulandı. Dosyaları istediğim nginx klasörüne almak kaldı. Burada raspi üzerindeki dosya yetkilendirmesini yönetici olarak değil o anki kullanıcının ismiyle yapmak lazım yoksa github kızıyor biraz. Bunu anlamak da yaklaşık 1 saat sürdüğüne göre aslında sorunun çözümü 

```shell
git clone git@github.com:Kullanıcı/repo.git
```

komutunun başına koyduğum ALIŞKANLIK olmuş olan "sudo" olduğunu fark etmem ise bir 30 dakika sürdü. Sonunda ama nginx'in /var/www/html dosyasına clonelamayı başardım. Ancak bir sorun var

-bu arada kod yazılımlarını nasıl çözdüm daha şık daha güzel duruyor di mi-

Bütün repoyu alıyorum bütün repodan spesifik bir yeri almam lazım çünkü site orada. Çözümü basit:

```shell
git clone --branch <branchname> <remote-repo-url>
```

bu kod ile hızlıca hallediyoruuuz.

Şimdi sıra nginx ayarında

nginx ayarı nispeten daha kolay çünkü uğraşmadan sadece doğru yerlere dosya yollarını belirtiyorsun ki bu yaptığım onca şey arasında en kolayı.

```shell
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
#
# In most cases, administrators will remove this file from sites-enabled/ and
# leave it as reference inside of sites-available where it will continue to be
# updated by the nginx packaging team.
#
# This file will automatically load configuration files provided by other
# applications, such as Drupal or Wordpress. These applications will be made
# available underneath a path with that package name, such as /drupal8.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
server {
	listen 80 default_server;
	listen [::]:80 default_server;

	# SSL configuration
	#
	# listen 443 ssl default_server;
	# listen [::]:443 ssl default_server;
	#
	# Note: You should disable gzip for SSL traffic.
	# See: https://bugs.debian.org/773332
	#
	# Read up on ssl_ciphers to ensure a secure configuration.
	# See: https://bugs.debian.org/765782
	#
	# Self signed certs generated by the ssl-cert package
	# Don't use them in a production server!
	#
	# include snippets/snakeoil.conf;

	root /var/www/html;

	# Add index.php to the list if you are using PHP
	index index.html index.htm index.nginx-debian.html;

	server_name _;

	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}

	# pass PHP scripts to FastCGI server
	#
	#location ~ \.php$ {
	#	include snippets/fastcgi-php.conf;
	#
	#	# With php-fpm (or other unix sockets):
	#	fastcgi_pass unix:/run/php/php7.4-fpm.sock;
	#	# With php-cgi (or other tcp sockets):
	#	fastcgi_pass 127.0.0.1:9000;
	#}

	# deny access to .htaccess files, if Apache's document root
	# concurs with nginx's one
	#
	#location ~ /\.ht {
	#	deny all;
	#}
}


# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#	listen 80;
#	listen [::]:80;
#
#	server_name example.com;
#
#	root /var/www/example.com;
#	index index.html;
#
#	location / {
#		try_files $uri $uri/ =404;
#	}
#}
```

Görüldüğü gibi # kaldır gerekli yerleri doldur. Basit sonrasında bütün bu işleri otomatikleştirme var.
Onlar da sırasıyla:

```Shell
sudo chmod 755 /var/www/html/repository-name
```

Yazma iznini verebilmek için

```Shell
git pull origin <branch ismi>
```

Githubtaki son güncellemeleri alsın diye

Test aşaması içinse sırasıyla 

```Shell
nginx -t
systemctl reload nginx
```

Otomasyon içinse 

```Shell
*/30 * * * * cd /path/to/your/nginx/website && git pull origin main > /var/log/cron_gitpull.log 2>&1
```

gibi ucube bir komut kullanıyorum. Çalışacağından kesinlikle emin değilim çalışmazsa başka yöntemler deneyeceğim. Bu adımlar tamam sırada bütün işlemlerin otomasyonu var. O da şu muazzam scriptle oluyor:

```Shell
# PowerShell Script for Windows

# Set variables for Obsidian to Hugo copy
$sourcePath = "C:\Users\path\to\obsidian\posts"
$destinationPath = "C:\Users\path\to\hugo\posts"

# Set Github repo 
$myrepo = "reponame"

# Set error handling
$ErrorActionPreference = "Stop"
Set-StrictMode -Version Latest

# Change to the script's directory
$ScriptDir = Split-Path -Parent $MyInvocation.MyCommand.Definition
Set-Location $ScriptDir

# Check for required commands
$requiredCommands = @('git', 'hugo')

# Check for Python command (python or python3)
if (Get-Command 'python' -ErrorAction SilentlyContinue) {
    $pythonCommand = 'python'
} elseif (Get-Command 'python3' -ErrorAction SilentlyContinue) {
    $pythonCommand = 'python3'
} else {
    Write-Error "Python is not installed or not in PATH."
    exit 1
}

foreach ($cmd in $requiredCommands) {
    if (-not (Get-Command $cmd -ErrorAction SilentlyContinue)) {
        Write-Error "$cmd is not installed or not in PATH."
        exit 1
    }
}

# Step 1: Check if Git is initialized, and initialize if necessary
if (-not (Test-Path ".git")) {
    Write-Host "Initializing Git repository..."
    git init
    git remote add origin $myrepo
} else {
    Write-Host "Git repository already initialized."
    $remotes = git remote
    if (-not ($remotes -contains 'origin')) {
        Write-Host "Adding remote origin..."
        git remote add origin $myrepo
    }
}

# Step 2: Sync posts from Obsidian to Hugo content folder using Robocopy
Write-Host "Syncing posts from Obsidian..."

if (-not (Test-Path $sourcePath)) {
    Write-Error "Source path does not exist: $sourcePath"
    exit 1
}

if (-not (Test-Path $destinationPath)) {
    Write-Error "Destination path does not exist: $destinationPath"
    exit 1
}

# Use Robocopy to mirror the directories
$robocopyOptions = @('/MIR', '/Z', '/W:5', '/R:3')
$robocopyResult = robocopy $sourcePath $destinationPath @robocopyOptions

if ($LASTEXITCODE -ge 8) {
    Write-Error "Robocopy failed with exit code $LASTEXITCODE"
    exit 1
}

# Step 3: Process Markdown files with Python script to handle image links
Write-Host "Processing image links in Markdown files..."
if (-not (Test-Path "images.py")) {
    Write-Error "Python script images.py not found."
    exit 1
}

# Execute the Python script
try {
    & $pythonCommand images.py
} catch {
    Write-Error "Failed to process image links."
    exit 1
}

# Step 4: Build the Hugo site
Write-Host "Building the Hugo site..."
try {
    hugo
} catch {
    Write-Error "Hugo build failed."
    exit 1
}

# Step 5: Add changes to Git
Write-Host "Staging changes for Git..."
$hasChanges = (git status --porcelain) -ne ""
if (-not $hasChanges) {
    Write-Host "No changes to stage."
} else {
    git add .
}

# Step 6: Commit changes with a dynamic message
$commitMessage = "New Blog Post on $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')"
$hasStagedChanges = (git diff --cached --name-only) -ne ""
if (-not $hasStagedChanges) {
    Write-Host "No changes to commit."
} else {
    Write-Host "Committing changes..."
    git commit -m "$commitMessage"
}

# Step 7: Push all changes to the main branch
Write-Host "Deploying to GitHub Master..."
try {
    git push origin master
} catch {
    Write-Error "Failed to push to Master branch."
    exit 1
}

# Step 8: Push the public folder to the hostinger branch using subtree split and force push
Write-Host "Deploying to GitHub Hostinger..."

# Check if the temporary branch exists and delete it
$branchExists = git branch --list "hostinger-deploy"
if ($branchExists) {
    git branch -D hostinger-deploy
}

# Perform subtree split
try {
    git subtree split --prefix public -b hostinger-deploy
} catch {
    Write-Error "Subtree split failed."
    exit 1
}

# Push to hostinger branch with force
try {
    git push origin hostinger-deploy:hostinger --force
} catch {
    Write-Error "Failed to push to hostinger branch."
    git branch -D hostinger-deploy
    exit 1
}

# Delete the temporary branch
git branch -D hostinger-deploy

Write-Host "All done! Site synced, processed, committed, built, and deployed."
```

Bu scripti kendim hazırlamadım o yüzden bazı şeylerin çalışmaması normal ama en azındaaaan

![[Screenshot 2024-12-28 095544.png]]

İLK GÜNÜN ÇALIŞTIĞINI GÖREBİLİYORUM. Nedense bugün geçmedi terminalde bazı hatalar var onları da yarın bakarım. Bu işin daha kolay yolu hosting almam olacak sanırım ama o kısım için beklemek durumundayım bugünlük paydos.

<script src="https://giscus.app/client.js"
        data-repo="Rezillique/my-notes"
        data-repo-id="R_kgDONjvPNw"
        data-category="Announcements"
        data-category-id="DIC_kwDONjvPN84ClpQq"
        data-mapping="pathname"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="dark"
        data-lang="en"
        crossorigin="anonymous"
        async>
</script>