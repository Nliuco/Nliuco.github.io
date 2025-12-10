# 1. 光标皮肤
## [Bibata Cursor](https://github.com/ful1e5/Bibata_Cursor)
```bash
paru -S bibata-cursor-theme-bin
```
# 2. 输入法皮肤(Fcitx5)
## [Ori theme](https://github.com/Reverier-Xu/Ori-fcitx5)
```bash
paru -S fcitx5-skin-ori-git
```
# 3. Kvantum安装
Kvantum 是一个基于 SVG 的 Qt 风格主题引擎（Qt Theme Engine）。
- 它的特点包括：
  - 使用 SVG 绘制界面组件 → 能实现精致的透明、模糊、圆角
  - 可加载第三方主题 → Kvantum 生态极其丰富（Catppuccin、Fluent、Nordic 等）
  - 不依赖 KDE Plasma → 所有 Qt 应用界面均生效
  - 能实现 GTK 样式做不到的毛玻璃效果
- 简而言之：
  - 系统外观靠 Plasma 主题；应用界面外观靠 Kvantum。
  - 想要透明 / 毛玻璃，就必须 Kvantum 出手。
```zsh
sudo pacman -S kvantum
```
# 4. [全局主题](https://github.com/Rudraksh88/zephyr-kvantum)
## 1. 克隆仓库
```zsh
# Create if it doesn't exist else skip
mkdir -p ~/.config/Kvantum
git clone https://github.com/Rudraksh88/zephyr-kvantum.git ~/.config/Kvantum/Zephyr
```
## 2. 设置主题
  - 打开 Kvantum 管理器 。
  - 在 Kvantum 管理器中，单击 “更改/删除主题” 选项卡。
  - 从可用主题列表中选择 Zephyr 。
  - 点击 “应用” 以设置主题。
# 5. [窗口装饰栏](https://github.com/Rudraksh88/KustomBreezeEnhanced)
## 步骤 1：安装依赖项
```zsh
sudo pacman -S base-devel
sudo pacman -S kdecoration qt6-declarative
sudo pacman -S cmake extra-cmake-modules
```
## 步骤 2：编译和安装
  ### 简易模式™ – 带脚本
  ```zsh
  chmod +x install.sh
  ./install.sh
  ```
  ### 手动模式
  ```zsh
  mkdir build && cd build
  cmake .. -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release -DKDE_INSTALL_LIBDIR=lib -DBUILD_TESTING=OFF -DKDE_INSTALL_USE_QT_SYS_PATHS=ON
  make
  sudo make install
  ```
# 6. 常用面板小组件
## 1. [Thermal Monitor](https://store.kde.org/p/998915/)
- Thermal Monitor 用于监控计算机的多个温度传感器，比如 CPU、GPU、硬盘 (HDD) 等 — 如果系统支持这些传感器。 
- 可以将它添加到 KDE 的面板 (panel) 或桌面上，从而实时显示温度数据。
## 2. [Minimal Chaac Weather](https://store.kde.org/p/2136307/)
- Minimal Chaac Weather for panel Plasma6
- Minimal Chaac Weather 显示当前天气和未来几天的天气预报。
- 它使用 Open‑Meteo API 来获取天气数据，然后通过 Bash 脚本处理并在 KDE 上展示。
- 插件“几乎不需要任何设置 (with no setup required)”，有用户表示安装后直接可用 — 很适合希望在桌面/面板上快速添加天气功能的人。
## 3. [Weather Widget Plus](https://store.kde.org/p/2281196)
- Weather Widget Plus 会在桌面或面板上显示天气信息，包括 “meteogram”（湿度 / 气温 / 气压 / 风向等图形化天气预报图表）和天气预报。
- 支持多种天气数据来源 (如 Open-Meteo, OpenWeather, Norwegian Meteorological Institute 等) —— 这意味着在很多地区都能拿到天气数据。
- 它是基于一个较老 widget 的分支 (fork)——即 “weather-widget-2”（原作者 blackadderkate），因此继承了一些稳定代码，同时继续由当前维护者 (tully-t) 更新 / 修正。
- 在使用 KDE Plasma 且希望在桌面或任务栏上方便地看到天气 (包括未来预报 + 图形 meteogram)，Weather Widget Plus 是一个不错、轻量而且灵活的选择 —— 特别适合经常关注天气 (比如通勤、出门、户外活动) 的用户。
## 4. [Netspeed Widget](https://store.kde.org/p/2136505)
- Netspeed Widget 的功能是“显示当前网络带宽 (bandwidth)／网络速度 (upload/download)”，也就是在桌面或面板 (panel) 上直观地显示你当前网络的上下行速度／带宽使用情况。
## 5. [Modern Clock](https://store.kde.org/p/1779868)
- Modern Clock 是一个高度可定制的时钟插件，旨在取代 KDE Plasma 中的默认时钟，提供更具现代感的视觉效果。
- 它允许用户选择不同的时钟样式（例如：数字、模拟、文本时钟等），并提供额外的定制选项，比如时区、时间格式、日期显示等。
- 插件可以直接集成到 KDE Plasma 面板或桌面，显示时间和日期。

