---
title: 'picoCTF 2024 | Binary Search WriteUp'
author: 'appdone'
categories: [picoCTF 2024 Challenges]
render_with_liquid: false
---

Bu yazıda, picoCTF platformunda yer alan "Binary Search" isimli meydan okumayı çözeceğiz. Meydan okuma sıcak soğuk oyunu oynayarak bayrağı bulmamızı istiyor. Bunun haricinde kaynak kodları verilmiş ama işimize yaramayacak gibi görünüyor.

```py
#!/bin/bash

target=$(( (RANDOM % 1000) + 1 ))

echo "Welcome to the Binary Search Game!"
echo "I'm thinking of a number between 1 and 1000."

trap 'echo "Exiting is not allowed."' INT
trap '' SIGQUIT
trap '' SIGTSTP

MAX_GUESSES=10
guess_count=0

while (( guess_count < MAX_GUESSES )); do
      read -p "Enter your guess: " guess

      if ! [[ "$guess" =~ ^[0-9]+$ ]]; then
          echo "Please enter a valid number."
          continue
      fi

      (( guess_count++ ))

      if (( guess < target )); then
          echo "Higher! Try again."
      elif (( guess > target )); then
          echo "Lower! Try again."
       else
          echo "Congratulations! You guessed the correct number: $target"

          flag=$(cat /challenge/metadata.json | jq -r '.flag')
          echo "Here's your flag: $flag"
          exit 0  # Exit with success code
       fi
done

echo "Sorry, you've exceeded the maximum number of guesses."
exit 1  # Exit with error code to close the connection
```

SSH servisine bağlandıktan sonra oyun başlıyor. Rastgele bir sayı seçiyor ve sıcak soğuk oyununu oynatarak bu sayıyı bulmamızı istiyor. Sayıyı bulduğumuzda ise bize bayrağı veriyor.

```
C:\Users\Appdone>ssh -p 57893 ctf-player@atlas.picoctf.net
ctf-player@atlas.picoctf.net's password:
Welcome to the Binary Search Game!
I'm thinking of a number between 1 and 1000.
Enter your guess: 800
Higher! Try again.
Enter your guess: 900
Lower! Try again.
Enter your guess: 860
Higher! Try again.
Enter your guess: 880
Higher! Try again.
Enter your guess: 890
Lower! Try again.
Enter your guess: 885
Higher! Try again.
Enter your guess: 886
Higher! Try again.
Enter your guess: 889
Congratulations! You guessed the correct number: 889
Here's your flag: picoCTF{***}
Connection to atlas.picoctf.net closed.
```
