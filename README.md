# Parsing_Serengeti-Park
In this notebook, we use Selenium and BeautifulSoup to **fully scrape the website Serengeti-Park.de**, including menu navigation, collecting all unique text, and clicking on interactive elements like "Mehr" buttons.  
The goal is to **train a bot** so it understands the website's structure and knows where each piece of information is located.


# Script
from selenium.webdriver import Chrome
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import NoSuchElementException, TimeoutException
from selenium.webdriver.common.by import By
from bs4 import BeautifulSoup
from time import sleep
from tqdm import tqdm
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
import time

service = Service("C:/Users/Luda/Downloads/chromedriver-win64/chromedriver-win64/chromedriver.exe")
browser = Chrome(service=service)
browser.maximize_window()

url = "https://serengeti-park.de/aktuelles"
browser.get(url)
time.sleep(5)  

browser_title = browser.title
print(f"Title of the page: {browser_title}")

soup = BeautifulSoup(browser.page_source, "lxml")
raw_menu = soup.find('div', class_='items-center justify-end hidden w-full lg:flex lg:w-auto lg:order-1').text
menu_items = [line.strip() for line in raw_menu.splitlines() if line.strip()]
print(f"Menu items: {menu_items}")

menu_div = soup.find('div', class_='items-center justify-end hidden w-full lg:flex lg:w-auto lg:order-1')
menu_links = menu_div.find_all('a') if menu_div else []

menu_items_link = []
for link in menu_links:
    text = link.text.strip()
    href = link.get('href')
    if text:  # leere überspringen
        menu_items_link.append({'title': text, 'url': href})
for item in menu_items_link:
    print(f"Menu item: {item['title']} -> URL: {item['url']}")

first_link = menu_items_link[0]
first_url = first_link['url']
browser.get(first_url)
WebDriverWait(browser, 10).until(EC.presence_of_element_located((By.TAG_NAME, 'body')))
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
header_element = WebDriverWait(browser, 10).until(
    EC.presence_of_element_located((By.XPATH, "//h2[contains(@class, 'font-head') and contains(@class, 'font-bold') and contains(@class, 'text-sp_light')]")))
print("Home", menu_items_link[0] )
header_text_home = header_element.text
print(header_text_home)
element = WebDriverWait(browser, 10).until(
    EC.presence_of_element_located((By.XPATH, "/html/body/main/div/div[4]/section/div/div/div/div/div[1]")))
element_text_home = element.text
print(element_text_home)

print('Entdecken', 'Aktuelles', 'Tierpark', 'Freizeitpark', 'VIP-Safaris', 'Serengeti-Park App')
def get_unique_text():
    elements = browser.find_elements(By.XPATH, "//*")
    seen = set()
    lines = []
    for el in elements:
        text = el.text.strip()
        if text:
            for line in text.split("\n"):
                clean_line = line.strip()
                if clean_line and clean_line not in seen:
                    seen.add(clean_line)
                    lines.append(clean_line)
    return lines
for i in range(1, 6):
    item = menu_items_link[i]
    print(f"\n--- {item['title']} -> {item['url']} ---\n")
    browser.get(item["url"])
    time.sleep(3)
    lines = get_unique_text()
    for line in lines:
        print(line)
    try:
        buttons = browser.find_elements(By.TAG_NAME, "button")
        mehr_buttons = [btn for btn in buttons if "Mehr" in btn.text]
        print(f"\n'Mehr' Buttons gefunden: {len(mehr_buttons)}")
        for idx, btn in enumerate(mehr_buttons):
            print(f"\n--- Klicke auf Button {idx+1}: {btn.text} ---")
            try:
                btn.click()
                time.sleep(3)
                lines = get_unique_text()
                for line in lines:
                    print(line)
            except Exception as e:
                print(f"Fehler beim Klicken auf Button: {e}")
            finally:
                browser.back()
                time.sleep(3)
    except Exception as e:
        print(f"Fehler bei der Suche nach Buttons: {e}")

browser.back()
time.sleep(3)

print('Besuchen', 'Tickets', 'Gruppenangebote', 'Öffnungszeiten', 'Anreise', 'FAQ')
exclude_btn_texts = [
    "ANMELDEN", "Absenden", "Route berechnen", "Download Parkplan", "Finde die passende VIP-Safari"]
def get_unique_text():
    elements = browser.find_elements(By.XPATH, "//*")
    seen = set()
    lines = []

    for el in elements:
        text = el.text.strip()
        if text:
            for line in text.split("\n"):
                clean_line = line.strip()
                if clean_line and clean_line not in seen:
                    seen.add(clean_line)
                    lines.append(clean_line)
    return lines

for i in range(6, 11):
    item = menu_items_link[i]
    print(f"\n--- {item['title']} -> {item['url']} ---\n")
    browser.get(item["url"])
    time.sleep(3)
    lines = get_unique_text()
    for line in lines:
        print(line)
    buttons = browser.find_elements(By.TAG_NAME, "button")
    valid_buttons = [btn for btn in buttons if btn.text.strip() and btn.text.strip() not in exclude_btn_texts]
    print(f"\nButtons gefunden: {len(valid_buttons)}")
    original_url = browser.current_url
    for idx, btn in enumerate(valid_buttons):
        try:
            btn_text = btn.text.strip()
            print(f"\n--- Klicke auf Button {idx + 1}: {btn_text} ---")
            browser.execute_script("arguments[0].scrollIntoView(true);", btn)
            time.sleep(1)
            browser.execute_script("arguments[0].click();", btn)
            time.sleep(3)
            new_url = browser.current_url
            lines = get_unique_text()
            for line in lines:
                print(line)
            if new_url != original_url:
                browser.back()
                time.sleep(3)
        except Exception as e:
            print(f"Fehler beim Klicken auf Button: {e}")
            continue

browser.back()
time.sleep(3)

print('Feiern', 'Shows & Events', 'Tagen &  Feiern')
exclude_btn_texts = [
    "ANMELDEN", "Absenden", "Download Kinderfreikarte 2025", "Download Gruppenangebote 2025"]
def get_unique_text():
    elements = browser.find_elements(By.XPATH, "//*")
    seen = set()
    lines = []
    for el in elements:
        text = el.text.strip()
        if text:
            for line in text.split("\n"):
                clean_line = line.strip()
                if clean_line and clean_line not in seen:
                    seen.add(clean_line)
                    lines.append(clean_line)
    return lines
for i in range(11, 13):
    item = menu_items_link[i]
    print(f"\n--- {item['title']} -> {item['url']} ---\n")
    browser.get(item["url"])
    time.sleep(3)
    lines = get_unique_text()
    for line in lines:
        print(line)
    buttons = browser.find_elements(By.TAG_NAME, "button")
    valid_buttons = [btn for btn in buttons if btn.text.strip() and btn.text.strip() not in exclude_btn_texts]
    print(f"\nButtons gefunden: {len(valid_buttons)}")
    original_url = browser.current_url
    for idx, btn in enumerate(valid_buttons):
        try:
            btn_text = btn.text.strip()
            print(f"\n--- Klicke auf Button {idx + 1}: {btn_text} ---")
            browser.execute_script("arguments[0].scrollIntoView(true);", btn)
            time.sleep(1)
            browser.execute_script("arguments[0].click();", btn)
            time.sleep(3)
            new_url = browser.current_url
            lines = get_unique_text()
            for line in lines:
                print(line)
            if new_url != original_url:
                browser.back()
                time.sleep(3)
        except Exception as e:
            print(f"Fehler beim Klicken auf Button: {e}")
            continue

browser.back()
time.sleep(3)

print('Übernachten')
# Ausschlussliste
exclude_btn_texts = ["ANMELDEN", "Zur Lodge-Buchung", "Download Kinderfreikarte 2025", "Gutschein bestellen"]
def get_unique_text():
    elements = browser.find_elements(By.XPATH, "//*")
    seen = set()
    lines = []
    for el in elements:
        text = el.text.strip()
        if text:
            for line in text.split("\n"):
                clean_line = line.strip()
                if clean_line and clean_line not in seen:
                    seen.add(clean_line)
                    lines.append(clean_line)
    return lines
original_url = "https://serengeti-park.de/uebernachten"
print(f"\n--- Übernachten -> {original_url} ---\n")
browser.get(original_url)
time.sleep(3)
lines = get_unique_text()
for line in lines:
    print(line)
all_buttons = browser.find_elements(By.TAG_NAME, "button")
target_buttons = [btn for btn in all_buttons if btn.text.strip() == "Jetzt buchen"]
print(f"\n'Jetzt buchen' Buttons gefunden: {len(target_buttons)}")
for btn_index in range(len(target_buttons)):
    print(f"\n--- Klicke auf Hauptbutton {btn_index + 1}: Jetzt buchen ---")
    try:
        browser.get(original_url)
        time.sleep(3)
        all_buttons = browser.find_elements(By.TAG_NAME, "button")
        target_buttons = [btn for btn in all_buttons if btn.text.strip() == "Jetzt buchen"]
        main_btn = target_buttons[btn_index]
        browser.execute_script("arguments[0].scrollIntoView(true);", main_btn)
        time.sleep(1)
        browser.execute_script("arguments[0].click();", main_btn)
        time.sleep(3)
        lines = get_unique_text()
        for line in lines:
            print(line)
        sub_buttons = browser.find_elements(By.TAG_NAME, "button")
        sub_buttons = [btn for btn in sub_buttons if btn.text.strip() and btn.text.strip() not in exclude_btn_texts]
        print(f"\n    Untergeordnete Buttons gefunden: {len(sub_buttons)}")
        for jdx, sub_btn in enumerate(sub_buttons):
            try:
                sub_text = sub_btn.text.strip()
                print(f"    -> Klicke auf untergeordneten Button {jdx + 1}: {sub_text}")
                browser.execute_script("arguments[0].scrollIntoView(true);", sub_btn)
                time.sleep(1)
                browser.execute_script("arguments[0].click();", sub_btn)
                time.sleep(2)
                lines = get_unique_text()
                for line in lines:
                    print("    " + line)
            except Exception as e:
                print(f"    !! Fehler beim untergeordneten Button: {e}")
                continue
    except Exception as e:
        print(f"!! Fehler beim Klicken auf 'Jetzt buchen' Button: {e}")
        continue

browser.back()
time.sleep(3)

exclude_btn_texts = [
    "ANMELDEN", "Zur Lodge-Buchung", "Download Kinderfreikarte 2025", "Gutschein bestellen",
    "Absenden", "Mehr laden", "Bestellen", "Konferenz 2025",
    "Praktikumsprogramm", "Zusammenfassungen der Praktikumsprojekte", "Unterstützte Projekte",
    "Aktuelle Publikationen", "Formulare", "Archiv Projekte", "Archiv Publikationen"]
for i in [14, 17]:
    item = menu_items_link[i]
    print(f"\n=== Navigiere zur Seite: {item['title']} -> {item['url']} ===\n")
    browser.get(item["url"])
    time.sleep(3)
    lines = get_unique_text()
    for line in lines:
        print(line)
    main_button_index = 0
    visited_buttons = set()

    while True:
        main_buttons = get_valid_buttons(on_main_page=True)
        if main_button_index >= len(main_buttons):
            break 
        try:
            btn = main_buttons[main_button_index]
            btn_text = btn.text.strip()
            if btn_text in exclude_btn_texts:
                print(f"    - Button überspringen: {btn_text}")
                main_button_index += 1
                continue
            if "Mehr erfahren" in btn_text and not btn.is_enabled():
                print(f"    - 'Mehr erfahren' Button ignorieren: {btn_text}")
                main_button_index += 1
                continue
            visited_buttons.add(btn_text)

            print(f"\n--- Klicke auf Hauptbutton {main_button_index + 1}: {btn_text} ---")
            browser.execute_script("arguments[0].scrollIntoView(true);", btn)
            time.sleep(1)
            browser.execute_script("arguments[0].click();", btn)
            time.sleep(3)
            lines = get_unique_text()
            for line in lines:
                print(line)
            inner_buttons = get_valid_buttons(on_main_page=False)
            print(f"    Verschachtelte Buttons: {len(inner_buttons)}")

            for idx, inner_btn in enumerate(inner_buttons):
                try:
                    inner_text = inner_btn.text.strip()
                    print(f"    -> Verschachtelter Button {idx + 1}: {inner_text}")
                    browser.execute_script("arguments[0].scrollIntoView(true);", inner_btn)
                    time.sleep(1)
                    browser.execute_script("arguments[0].click();", inner_btn)
                    time.sleep(2)

                    inner_lines = get_unique_text()
                    for line in inner_lines:
                        print("    " + line)

                except Exception as e:
                    print(f"    !! Fehler beim verschachtelten Button: {e}")
                    continue
            print(f"\n<<< Zurück zur Seite: {item['url']} >>>\n")
            browser.get(item["url"])
            time.sleep(3)

        except Exception as e:
            print(f"!! Fehler beim Klicken auf den Hauptbutton: {e}")
            continue
        main_button_index += 1

    print(f"\n>>> Weiter zur nächsten URL in der Liste: {menu_items_link[i+1]['url']} <<<")

## Screenshots
![1](https://github.com/user-attachments/assets/eed3c16e-6fba-441b-9cd9-249d28ca0e08)
![2](https://github.com/user-attachments/assets/7d14fa1a-b249-49b9-b336-a07a06428ae6)
![3](https://github.com/user-attachments/assets/c154e701-09b9-45d8-a1d0-e7302b93f681)
![4](https://github.com/user-attachments/assets/93966208-0e61-4119-b94f-b7f7671471ac)
![5](https://github.com/user-attachments/assets/42a046ca-0513-4eb4-8707-4f6945eda59e)




