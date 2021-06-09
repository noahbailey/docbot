# QR Codes 

## Scan a QR with CLI

Install the zbar tool: 

    sudo pacman -S zbar

Scan a QR from an image file: 

    zbarimg ~/Desktop/qrcode.png

## Create a QR code with CLI

Install qrencode: 

    sudo pacman -S qrencode

Generate a QR code: 

    echo "https://www.youtube.com/watch?v=dQw4w9WgXcQ" | qrencode -s 6 -l H -o output.png

Print the QR code: 

    echo "https://www.youtube.com/watch?v=dQw4w9WgXcQ" | qrencode -t utf8i -m2 -l H | lpr
