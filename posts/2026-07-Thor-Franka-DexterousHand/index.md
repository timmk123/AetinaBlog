---
title: "用 NVIDIA Isaac ROS 在 Jetson Thor 上控制 Franka 手臂與 Axis 靈巧手：一套從感知到執行的具身 AI 系統"
date: 2026-07-24
status: draft   # draft | published
tags: ["RobotArm", "IsaacROS", "JetsonThor", "ROS", "Franka", "DexterousHand", "EmbodiedAI"]
exhibition: "DigitalAvatar"
---

## 前言

在 Computex 2025 與 2026 的展場中，安提展出了 Digital Human 搭配機械手臂互動的系統。站在展台前的觀眾，不只看到一個會說話的數位人，還看到一隻真實的機器手臂在身旁同步動作——這背後，是由跑在 **NVIDIA Jetson Thor** 上的 **Isaac ROS**、**Franka 機械手臂**，以及 **Axis 靈巧手**共同構成的具身 AI 系統。

這篇文章將介紹這套系統的組成邏輯與整合方式，說明我們如何把 NVIDIA Isaac ROS 與實體硬體串接在一起，讓 AI 的輸出不只停留在螢幕上，而是真正透過機械手臂延伸到物理世界。

---

## NVIDIA Isaac ROS × Jetson Thor 的角色

這套系統由兩個 NVIDIA 核心元件共同撐起：**Isaac ROS** 是軟體層，**Jetson Thor** 是硬體平台。

**Isaac ROS** 是 NVIDIA 推出的 GPU 加速 ROS 2 套件集合，將 NVIDIA 的 AI 推論能力直接整合進 ROS 2 的開發生態。它提供了一系列針對機器人感知、操作、導航優化的 ROS 2 package，讓開發者能夠在熟悉的 ROS 介面下，直接調用 GPU 加速的深度學習模型，不需要在 ROS 與推論框架之間另外建立橋接層。

**Jetson Thor** 則是 NVIDIA 最新一代的邊緣 AI 運算平台，專為具身 AI 與機器人應用設計。相較於前代 Jetson，Thor 大幅提升了 AI 推論算力，能夠在邊緣端本地運行複雜的感知與控制模型，不依賴雲端，這對展場這種對即時性要求高、網路環境不穩定的場合來說，是非常關鍵的特性。

兩者的組合，讓整套機器人控制系統可以在一台緊湊的邊緣裝置上完成感知、推論與控制的全流程。

---

## 系統架構：三者如何協作

這套展場系統的協作關係可以用三個層次來描述：

### 感知與決策層：Isaac ROS on Jetson Thor

Jetson Thor 搭載 Isaac ROS，位於整個系統的「大腦」位置。它接收來自相機或感測器的環境資訊，透過 GPU 加速的感知與操作模型推論出當前情境下手臂應該執行的動作序列，再將結果以 ROS 2 Topic 的形式輸出到下一層。

Isaac ROS 的設計讓系統不只是「執行固定指令」，而是能根據當下感知到的環境狀態做出適應性的動作判斷，這是讓整套系統看起來更自然、更智慧的關鍵。

### 通訊中介層：ROS 2

Isaac ROS 輸出的控制訊號，透過 **ROS 2** 的 Topic / Service 機制傳遞到各個硬體驅動端。ROS 2 在這裡扮演的是統一的通訊匯流排角色——不論是手臂的關節控制、靈巧手的手指開合，還是感測器的狀態回饋，都透過 ROS 2 的 Publisher / Subscriber 架構統一管理。

這樣的設計帶來極大的模組化彈性：每個硬體元件只需要實作自己的 ROS 2 Driver，就能與整個系統接軌，而不需要對每個元件個別客製化通訊協定。未來要替換或新增硬體，也只需要對應調整驅動層，上層的推論邏輯不需要改動。Isaac ROS 本身即是 ROS 2 的延伸，兩者天然整合，不需要額外的橋接轉換。

### 執行層：Franka 機械手臂 × Axis 靈巧手

**Franka 機械手臂**負責的是大範圍的空間移動——抬起、伸展、轉向等動作，提供整個系統的主要自由度。Franka 有完整的 ROS 2 支援（`franka_ros2`），可以透過 ROS 2 直接接受關節角度或末端位置的控制指令，與 Isaac ROS 的輸出直接對接。

**Axis 靈巧手**則接在 Franka 的末端，負責精細的手指動作。靈巧手的多自由度設計，讓系統能夠完成抓取、捏合、手勢等細膩操作，是讓整套系統從「工業機械臂」升級到「類人手部操作」的關鍵硬體。兩者透過 ROS 2 的統一介面協同運作，彼此的控制指令可以同步發送，實現手臂移動與手指動作的協調配合。

---

## 為什麼選擇 Isaac ROS + Jetson Thor

在整合這套系統的過程中，Isaac ROS 搭配 Jetson Thor 帶來了幾個具體優勢：

**Isaac ROS 即是 ROS 2，沒有額外學習成本**
Isaac ROS 是建立在標準 ROS 2 之上的延伸套件，對原本就使用 ROS 2 的機器人開發者來說，導入 Isaac ROS 幾乎沒有額外的學習門檻。開發者可以繼續使用熟悉的 Topic、Service、Launch 等 ROS 2 概念，同時直接獲得 NVIDIA GPU 加速的 AI 推論能力。

**與 NVIDIA 模擬生態深度整合**
Isaac ROS 與 NVIDIA Isaac Sim（模擬環境）高度整合。在 Isaac Sim 中訓練好的操作策略模型，可以透過 Isaac ROS 直接部署到實機上進行推論，大幅縮短從模擬到實體的移植成本（Sim-to-Real Gap）。

**Jetson Thor 的邊緣端 AI 算力**
Jetson Thor 提供了足以在邊緣端本地執行複雜 AI 模型的算力，整套系統不依賴雲端服務。這在展場環境中特別重要，確保展示系統的穩定性與即時性不受網路條件影響，同時也降低了對外部基礎設施的依賴風險。

**完整的 NVIDIA 具身 AI 生態系**
從硬體（Jetson Thor）、模擬（Isaac Sim）、訓練（Isaac Lab）到邊緣部署（Isaac ROS），NVIDIA 提供了一條完整的具身 AI 開發鏈，讓整個工作流程在同一個生態系內完成，減少跨平台整合的摩擦與維護成本。

---

## 展場中的實際呈現

在 Computex 展台上，這套系統與 Digital Human 整合在同一個互動情境中。當觀眾與 Digital Human 互動時，Digital Human 不只以語音和表情回應，Franka 手臂也會同步做出對應的手勢與動作，Axis 靈巧手則在適當的時機展示精細的手指操作，形成一個視覺上完整、行為上協調的展示系統。

整個展示的運算都在本地進行，由安提自有的 AI 伺服器方案承載：

| 展示主機 | GPU 配置 | 負責任務 |
| :--- | :--- | :--- |
| [Aetina AEX-2UA1](https://www.aetina.com/ch/products-detail.php?i=730) | NVIDIA RTX PRO™ 6000 Blackwell + NVIDIA® L4 | Digital Avatar 渲染 + Isaac ROS 推論 |
| [Aetina AIP-FR68S](https://www.aetina.com/ch/products-detail.php?i=727) | NVIDIA® L40S + NVIDIA® L4 | AI Agent 推論 + 機器人控制 |

<!-- TODO: 補上展場實際運作的照片 -->

---

## 小結

這套系統展示的，是具身 AI 從感知到執行的完整閉環：跑在 Jetson Thor 上的 Isaac ROS 負責感知與推論，ROS 2 負責統一調度，Franka 與 Axis 負責在物理世界落地執行。三個層次分工清晰，又透過 ROS 2 緊密整合，是目前在展場級應用中，能夠快速落地的具身 AI 整合方案之一。

下一篇文章，我們會進一步介紹如何將 Audio2Face 語音轉表情的技術串接到 Digital Human，讓展示系統的語音互動更加自然與即時。

---

## 參考資料

- [NVIDIA Isaac ROS](https://developer.nvidia.com/isaac/ros)
- [NVIDIA Jetson Thor](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-thor/)
- [NVIDIA Isaac Sim](https://developer.nvidia.com/isaac/sim)
- [NVIDIA Isaac Lab](https://developer.nvidia.com/isaac/lab)
- [franka_ros2](https://github.com/frankaemika/franka_ros2)

## 產品連結

- [Aetina AEX-2UA1](https://www.aetina.com/ch/products-detail.php?i=730)
- [Aetina AIP-FR68S](https://www.aetina.com/ch/products-detail.php?i=727)
