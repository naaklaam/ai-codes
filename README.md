#!/bin/bash

# Install dependensi di Termux
pkg install openssl steghide -y

# Fungsi enkripsi AES
encrypt() {
    echo -n "$1" | openssl enc -aes-256-cbc -a -salt -pass pass:"$2" 2>/dev/null
}

# Fungsi dekripsi AES
decrypt() {
    echo -n "$1" | openssl enc -d -aes-256-cbc -a -salt -pass pass:"$2" 2>/dev/null
}

# Menu utama
while true; do
    clear
    echo -e "\033[1;36m=== STEGANO + KRIPTOGRAFI ===\033[0m"
    echo "1. Sembunyikan Pesan (Enkripsi + Stegano)"
    echo "2. Ekstrak Pesan (Dekripsi + Stegano)"
    echo "3. Keluar"
    read -p "Pilih opsi: " choice

    case $choice in
        1)
            read -p "Masukkan pesan rahasia: " message
            read -p "Masukkan password enkripsi: " crypto_pass
            read -p "Masukkan password steganografi: " stego_pass
            read -p "Path gambar asli (contoh: /sdcard/selfie.jpg): " image_path
            
            # Enkripsi pesan
            encrypted_msg=$(encrypt "$message" "$crypto_pass")
            
            # Simpan pesan terenkripsi ke file sementara
            echo "$encrypted_msg" > encrypted.txt
            
            # Sisipkan ke gambar
            steghide embed -cf "$image_path" -ef encrypted.txt -p "$stego_pass" -f
            rm encrypted.txt
            
            echo -e "\033[1;32m[+] Pesan tersembunyi di ${image_path}\033[0m"
            ;;
        2)
            read -p "Path gambar berisi pesan: " image_path
            read -p "Password enkripsi: " crypto_pass
            read -p "Password steganografi: " stego_pass
            
            # Ekstrak pesan
            steghide extract -sf "$image_path" -p "$stego_pass" -xf extracted.txt -f
            
            # Dekripsi pesan
            encrypted_msg=$(cat extracted.txt)
            decrypted_msg=$(decrypt "$encrypted_msg" "$crypto_pass")
            rm extracted.txt
            
            echo -e "\033[1;32m[+] Pesan asli: $decrypted_msg\033[0m"
            ;;
        3)
            exit 0
            ;;
        *)
            echo -e "\033[1;31m[!] Pilihan tidak valid\033[0m"
            ;;
    esac
    read -p "Tekan Enter untuk melanjutkan..."
done
