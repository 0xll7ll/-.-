# simple_ping_scanner.py
# يعمل على Android / Pydroid3 بدون صلاحيات
# يفحص الأجهزة المتصلة باستخدام Ping

import os
import ipaddress
import subprocess
import platform
from concurrent.futures import ThreadPoolExecutor, as_completed
import csv
import time

# المسار لحفظ النتائج
CSV_FILENAME = "/storage/emulated/0/Download/ping_scan_results.csv"

def parse_targets(input_str):
    s = input_str.strip()
    if not s:
        return []
    if '/' in s:
        try:
            net = ipaddress.ip_network(s, strict=False)
            return [str(ip) for ip in net.hosts()]
        except Exception as e:
            print("خطأ في صيغة CIDR:", e)
            return []
    return [s]

def ping_host(ip):
    param = "-n" if platform.system().lower()=="windows" else "-c"
    try:
        result = subprocess.run(
            ["ping", param, "1", ip],
            stdout=subprocess.DEVNULL,
            stderr=subprocess.DEVNULL
        )
        return result.returncode == 0
    except Exception:
        return False

def save_csv(results):
    try:
        with open(CSV_FILENAME, "w", newline="", encoding="utf-8") as f:
            writer = csv.writer(f)
            writer.writerow(["IP", "Status"])
            for ip, status in results.items():
                writer.writerow([ip, "Online" if status else "Offline"])
        print(f"\n✅ تم حفظ النتائج في {CSV_FILENAME}")
    except Exception as e:
        print("⚠️ خطأ في حفظ الملف:", e)

def main():
    print("🔍 Simple Ping Network Scanner
