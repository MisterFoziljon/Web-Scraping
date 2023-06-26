### Web-Scraping

![asosiy](https://miro.medium.com/v2/resize:fit:1400/1*1QcqrOoDE1rKa0NTp1iEtw.png)

Web scraping - bu web-saytlardan ma'lumotlarni olish jarayoni. Bu tizimlashtirilgan tarzda web-sahifalardan ma'lumotlarni olish, tahlil qilish va chiqarish uchun avtomatlashtirilgan skriptlar yoki dasturiy vositalardan foydalanishni o'z ichiga oladi. Ma'lumotlar HTML, XML yoki JSON kabi turli formatlarda bo'lishi mumkin.

![data](https://i.pinimg.com/originals/38/87/f4/3887f4e1986e94fea6b7b2fbf7a2fbcb.png)

Web scraping odatda web saytga HTTP so'rovlarini yuborishni va keyin kerakli ma'lumotlarni olish uchun javobning HTML tarkibini tahlil qilishni o'z ichiga oladi. Bu matn, rasmlar, havolalar, jadvallar yoki veb-saytda mavjud bo'lgan boshqa ma'lumotlarni o'z ichiga olishi mumkin. Veb-qirqish bir nechta sahifalardan yoki turli veb-saytlardan ma'lumotlarni to'plash uchun ishlatilishi mumkin, bu foydalanuvchilarga tahlil, tadqiqot yoki boshqa maqsadlar uchun katta hajmdagi ma'lumotlarni to'plash imkonini beradi.

#### Scraping uchun kerakli kutubxonalar (environments):

🚀 [BeautifulSoup](https://pypi.org/project/beautifulsoup4/) version = 4.12.2

🚀 [Selenium](https://pypi.org/project/selenium/) version = 4.10.0

Kerakli kutubxonalarni import qilamiz

```python
from bs4 import BeautifulSoup 
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.firefox.options import Options
from selenium.webdriver.firefox.service import Service
from selenium.common.exceptions import NoSuchElementException
import pandas as pd
```

[Webdriver](https://medium.com/coderbyte/why-is-selenium-webdriver-a-popular-choice-for-automation-testing-3bc2aee57bff) sifatida gecodriver dan foydalanamiz. Web brauzer sifatida Firefox dan foydalanilgan. Webdriverning kerakli sozlamalari o'rnatilgan va web brauzer foydalanish uchun tayyor holatga keltirilgan.

```python
webdriver_path = 'geckodriver'
binary_path = "C:\Program Files (x86)\Mozilla Firefox\firefox.exe"
firefox_options = Options()
firefox_options.add_argument('--headless')
firefox_options.add_argument('--no-sandbox')
driver = webdriver.Firefox(service=Service(webdriver_path), options=firefox_options)
```

Yangiliklarni scrap qilish uchun [qalampir.uz](https://qalampir.uz) sayti tanlab olindi:

```python
url = "https://qalampir.uz"
```

Saytning o'zbek (lotin) tilidagi sahifalarini scrap qilamiz. Yangiliklar turli kategoriyalarda bo'lgani uchun barcha kategoriya linklarini quyidagi usulda olamiz:

```python
driver.get(url+"/uz")
html_content = driver.page_source
soup = BeautifulSoup(html_content, "html.parser")
items = soup.find_all("a",class_="nav-link")
menu = []

for item in items:
    menu.append(url+"/uz"+item["href"])
    print(url+"/uz"+item["href"])
```

Kategoriya nomlarini listga yig'ib olamiz:

```python
category_name = [category.split("/")[-1] for category in menu]
category_name
```

![category](https://github.com/MisterFoziljon/Web-Scraping/blob/main/rasmlar/category.png)

Har bir kategoriyaga tegishli bo'lgan barcha sahifalarni ochib olamiz. 
Buning uchun sahifa ostidagi ```Yana yuklash``` tugmachasini sahifalar to'liq ochilib bo'lguniga qadar bosib boramiz.

![category](https://github.com/MisterFoziljon/Web-Scraping/blob/main/rasmlar/button.png)

Har bir yangilik linki va kategoriyasini faylga saqlab boramiz.

```python
with open("qalampir_uz.txt", "+a") as file:
    
    separator = "~~~#~~~"
    
    for item, category in zip(menu, category_name):
        print(f"{item} scrap boshlandi...")
        driver.get(item)
        page = 0
        
        while True:
            page+=1
            
            try:
                element = driver.find_element(By.XPATH,"//button[@class=\"refresh-btn\"]")
                element = element.click()
                
                if page%100==0:
                    print(f"{page} ta sahifa load more qilindi")
                
            except NoSuchElementException:
                break
        
        print(f"{item} scrap tugadi\n\n")
        
        html_content = driver.page_source
        soup = BeautifulSoup(html_content, "html.parser")
        items = soup.find_all("a",class_="news-card")
        count = 0
        
        for item in items:
            href = url+item["href"]
            file.write(f"{href}{separator}{category}\n")
```

Ushbu sahifadan yangilikning sarlavhasini scrap qilib olamiz.
![category](https://github.com/MisterFoziljon/Web-Scraping/blob/main/rasmlar/title.png)

Bu sahifadan esa kontentni scrap qilib olamiz.
![category](https://github.com/MisterFoziljon/Web-Scraping/blob/main/rasmlar/p.png)

Bu yerda xuddi shu amallar bajarilishini ko'rishingiz mumkin:

```python
with open("qalampir.txt","+a") as wfile:
    with open("qalampir_uz.txt") as rfile:
        
        for items in rfile:
            link, category = items.split(separator)
            print(link)
            driver.get(link)
            html_content = driver.page_source
            soup = BeautifulSoup(html_content, "html.parser")

            title = soup.find("h1",class_="text").text # sarlavhani olish 
            content = "".join([p.text for p in soup.find("div", class_ = "row g-4 my-main-content").find_all("p")]) # content ni olish

            wfile.write(f"{title}{separator}{content}{separator}{category}\n")
```

Yig'ilgan dataset *.txt formatidan *.csv formatiga o'tkaziladi. Bunda pandas kutubxonasidagi DataFrame dan foydalanamiz.

```python
def clean_element(element):
    return element.replace("\n","").strip()


dataset = pd.DataFrame(columns=['title', 'content', 'category'])

with open("qalampir.txt") as file:
    for news in file:
        
        news = news.split(separator)
        dataset = dataset.append([{'title':clean_element(news[0]), 'content':clean_element(news[1]), 'category':clean_element(news[2])}],ignore_index = True)
        
dataset.head()
```

Hosil bo'lgan Data Frame ni csv holatida saqlaymiz.

```python
dataset.to_csv("qalampir.csv", encoding='utf-8', index = False)
```

```Web saytlarni scrap qilish paytida saytning xizmat ko'rsatish shartlarini hurmat qilish, huquqiy va axloqiy ko'rsatmalarga rioya qilish va mualliflik huquqi yoki ma'lumotlarni himoya qilish bo'yicha har qanday amaldagi qonunlarni yodda tutish kerak.```
