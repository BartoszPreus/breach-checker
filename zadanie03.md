Zadanie 1:
PS C:\Users\barte\Desktop\Bartus\studia\V semestr i III semestr\Cyberbezpieczeństwo\metody wirtualizacji\labolatoria\lab3 nauka do wejsciowki\breach-checker> docker inspect --format '{{json .Config.Labels}}' bartosz-frontend:v1    
{"autor":"Bartosz Preus","data":"2026-10-16","maintainer":"BartoszPreus@users.noreply.github.com"}
PS C:\Users\barte\Desktop\Bartus\studia\V semestr i III semestr\Cyberbezpieczeństwo\metody wirtualizacji\labolatoria\lab3 nauka do wejsciowki\breach-checker> docker inspect --format '{{json .Config.Labels}}' bartosz-frontend:latest
{"autor":"Bartosz Preus","data":"2026-10-16","maintainer":"BartoszPreus@users.noreply.github.com"}
