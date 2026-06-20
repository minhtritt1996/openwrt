# 🚀 Firmware OpenWrt tăng tốc phần cứng Dual-NSS cho Meraki MR52

Kho mã nguồn này chứa một phiên bản OpenWrt được tùy biến và tối ưu hóa chuyên sâu dành riêng cho Cisco Meraki MR52 (IPQ8068).

Điểm nổi bật nhất của firmware này là bản vá Dual-NSS Hardware Acceleration, cho phép kích hoạt và khai thác toàn bộ sức mạnh của hai nhân Qualcomm Network Subsystem (NSS). Các tác vụ xử lý mạng sẽ được chuyển từ CPU chính sang bộ xử lý NSS chuyên dụng, giúp thiết bị có thể định tuyến lưu lượng Gigabit với mức sử dụng CPU dưới 5%.

## ✨ Tính năng nổi bật

**1. Tích hợp Dual NSS-GMAC (`qca-nss-gmac`)**
- Cả hai cổng mạng `eth0` (PoE) và `eth1` (Non-PoE) đều được ánh xạ trực tiếp tới trình điều khiển NSS.
- Loại bỏ nút thắt hiệu năng do driver Linux mặc định `stmmac`.
- Khóa các chân PHY Reset (GPIO 6 và GPIO 7) bằng cơ chế `gpio-hog`, tránh việc kernel Linux vô tình reset PHY trong quá trình đàm phán SGMII.

**2. Tăng tốc phần cứng ECM (`qca-nss-ecm`)**
- Offload hoàn toàn định tuyến IPv4 và IPv6 sang NSS.
- Hỗ trợ xử lý PPPoE bằng phần cứng.
- Tăng tốc SQM (Smart Queue Management) thông qua `sqm-scripts-nss`.
- Giảm đáng kể tải CPU khi xử lý lưu lượng mạng tốc độ cao.

**3. Bộ tăng tốc mã hóa phần cứng (`qca-nss-crypto`)**
- Kích hoạt bộ xử lý mã hóa tích hợp trên nền tảng Qualcomm IPQ806x.
- Tăng tốc IPsec bằng phần cứng (AES/SHA).
- Hỗ trợ tăng tốc OpenVPN thông qua Cryptodev (`/dev/crypto`).

**4. Tối ưu hóa nền tảng**
- Tinh chỉnh riêng cho Cisco Meraki MR52.
- Cải thiện độ ổn định của giao tiếp SGMII.
- Tích hợp firmware NSS được tối ưu.
- Giảm độ trễ và mức sử dụng CPU trong các tác vụ mạng nặng.

## 🛠️ Hướng dẫn biên dịch

**1. Cài đặt các gói phụ thuộc**
```bash
sudo apt update
sudo apt install -y build-essential clang flex bison g++ awk gcc-multilib g++-multilib \
gettext git libncurses5-dev libssl-dev python3-distutils rsync unzip zlib1g-dev \
file wget curl python3-setuptools python3-pip
```

**2. Tải mã nguồn**
```bash
git clone -b openwrt-23.05-nss-qsdk11 https://github.com/minhtritt1996/openwrt.git mr52-nss
cd mr52-nss
```

**3. Cập nhật Feeds**
```bash
./scripts/feeds update -a
./scripts/feeds install -a
```

**4. Cấu hình OpenWrt**
```bash
make menuconfig
```
Lựa chọn:
- Target System: **Qualcomm Atheros IPQ806X**
- Subtarget: **Generic**
- Target Profile: **Cisco Meraki MR52**

**5. Bắt đầu biên dịch**
```bash
make V=s -j1
```
Hoặc sử dụng toàn bộ luồng CPU:
```bash
make -j$(nproc)
```

**6. Firmware đầu ra**
Sau khi biên dịch thành công, firmware sẽ nằm tại:
```text
bin/targets/ipq806x/generic/openwrt-ipq806x-generic-meraki_mr52-squashfs-sysupgrade.bin
```

## 📊 Kiểm tra và chẩn đoán

**Theo dõi tải của NSS**
```bash
while true; do
    clear
    cat /sys/kernel/debug/qca-nss-drv/stats/cpu_load_ubi
    sleep 1
done
```

**Theo dõi số lượng gói tin được Offload**
```bash
while true; do
    clear
    cat /sys/kernel/debug/qca-nss-drv/stats/ipv4 | head -n 15
    sleep 1
done
```

**Theo dõi số kết nối được Offload**
```bash
while true; do
    clear
    cat /sys/kernel/debug/ecm/ecm_db/connection_count
    sleep 1
done
```

## 📈 Hiệu năng kỳ vọng

| Tính năng | OpenWrt tiêu chuẩn | Bản Dual-NSS |
| --- | --- | --- |
| **Định tuyến IPv4** | CPU xử lý | NSS xử lý |
| **Định tuyến IPv6** | CPU xử lý | NSS xử lý |
| **PPPoE** | CPU xử lý | NSS xử lý |
| **SQM QoS** | Tốn nhiều CPU | Tăng tốc bởi NSS |
| **VPN IPsec** | Một phần bằng phần mềm | Tăng tốc phần cứng |
| **Thông lượng Gigabit** | CPU tải cao | CPU dưới 5% |

---
*Phiên bản OpenWrt 23.05 NSS QSDK11 được tối ưu dành riêng cho Cisco Meraki MR52 với hỗ trợ đầy đủ Dual-NSS Hardware Acceleration.*
