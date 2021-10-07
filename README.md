# ############################################################ ##################################
# Lingkungan Bootstrap Termux.
DARI awal SEBAGAI bootstrap

ARG BOOTSTRAP_VERSION=2021.09.26-r1
ARG BOOTSTRAP_ARCH=i686
ARG SYSTEM_TYPE=x86

# Docker menggunakan /bin/sh secara default, tetapi saat ini kami tidak memilikinya.
SHELL [ "/system/bin/sh" , "-c" ]
ENV PATH /sistem/bin

# Salin libc, tautan, dan beberapa utilitas.
SALIN /sistem/$SYSTEM_TYPE /sistem

# Host DNS statis: karena sistem kami tidak memiliki resolver DNS, kami akan
# harus menyelesaikan domain secara manual dan mengisi /system/etc/hosts.
SALIN /static-dns-hosts.txt /system/etc/static-dns-hosts.txt

# Ekstrak arsip bootstrap dan buat symlink.
TAMBAHKAN https://github.com/termux/termux-packages/releases/download/bootstrap-$BOOTSTRAP_VERSION/bootstrap-$BOOTSTRAP_ARCH.zip /bootstrap.zip
RUN busybox mkdir -p /data/data/com.termux/files && \
    cd /data/data/com.termux/files && \
    busybox mkdir ../cache ./usr ./home && \
    busybox unzip -d usr /bootstrap.zip && \
    busybox rm /bootstrap.zip && \
    cd ./usr && \
    busybox cat SYMLINKS.txt | saat membaca -r baris; melakukan \
      dest=$(echo "$line" | busybox awk -F '←'  '{ print $1 }' ); \
      link=$(echo "$line" | busybox awk -F '←'  '{ print $2 }' ); \
      busybox ln -s "$dest"  "$link" ; \
    selesai && \
    busybox rm SYMLINKS.txt && \
    busybox ln -s /data/data/com.termux/files/usr /usr && \
    busybox ln -s /data/data/com.termux/files/usr/bin /bin && \
    busybox ln -s /data/data/com.termux/files/usr/tmp /tmp

# Atur mode kepemilikan dan akses file:
# * Konten pengguna dimiliki oleh 1000:1000.
# * Mode file termux hanya disetel untuk pengguna.
# * Istirahat dimiliki oleh root dan memiliki mode 755/644.
RUN busybox chown -Rh 0:0 /system && \
    busybox chown -Rh 1000:1000 /data/data/com.termux && \
    busybox chown 1000:1000 /system/etc/hosts /system/etc/static-dns-hosts.txt && \
    busybox find /system -type d -exec busybox chmod 755 "{}"  \; && \
    busybox find /system -type f -executable -exec busybox chmod 755 "{}"  \; && \
    busybox temukan /system -type f ! -executable -exec busybox chmod 644 "{}"  \; && \
    busybox find /data -type d -exec busybox chmod 755 "{}"  \; && \
    busybox temukan /data/data/com.termux/files -type f -o -type d -exec busybox chmod g-rwx,o-rwx "{}"  \; && \
    cd /data/data/com.termux/files/usr && \
    busybox temukan ./bin ./lib/apt ./lib/bash ./libexec -type f -exec busybox chmod 700 "{}"  \;

# Gunakan utilitas dari Termux dan alihkan pengguna ke non-root.
ENV PATH /data/data/com.termux/files/usr/bin
SHELL [ "/data/data/com.termux/files/usr/bin/sh" , "-c" ]
PENGGUNA 1000:1000

# Perbarui cache DNS statis saat login. Juga skrip symlink dan daftar host ke awalan.
RUN echo "echo -e 'Memperbarui DNS statis: \n ' && /system/bin/update-static-dns && echo" \
    > /data/data/com.termux/files/home/.bashrc && \
    ln -s /system/bin/update-static-dns /data/data/com.termux/files/usr/bin/update-static-dns && \
    ln -s /system/etc/static-dns-hosts.txt /data/data/com.termux/files/usr/etc/static-dns-hosts.txt

# Perbarui cache DNS statis, instal pembaruan, dan pembersihan.
JALANKAN /system/bin/update-static-dns && \
    tepat memperbarui && \
    apt upgrade -o Dpkg::Options::=--force-confnew -yq && \
    rm -rf /data/data/com.termux/files/usr/var/lib/apt/* && \
    rm -rf /data/data/com.termux/files/usr/var/log/apt/* && \
    rm -rf /data/data/com.termux/cache/apt/*

# ############################################################ ##################################
# Buat gambar akhir.
DARI awal

ENV ANDROID_DATA /data
ENV ANDROID_ROOT /sistem
ENV HOME /data/data/com.termux/files/home
ENV LANG en_US.UTF-8
ENV PATH /data/data/com.termux/files/usr/bin
ENV PREFIX /data/data/com.termux/files/usr
ENV TMPDIR /data/data/com.termux/files/usr/tmp
ENV TZ UTC

SALINAN --dari=bootstrap / /

WORKDIR /data/data/com.termux/files/home
SHELL [ "/data/data/com.termux/files/usr/bin/sh" , "-c" ]
PENGGUNA 1000:1000

CMD [ "/data/data/com.termux/files/usr/bin/login" ]
