---
layout: post
title: "FurnitureStyle：把圖片轉成可搜尋的家具描述"
description: "這個原型沒有用單一提示同時辨識家具與改寫顏色，而是把工作拆成兩個可檢查的步驟，再進行搜尋。"
date: 2026-07-24 01:02:00 +0800
permalink: /zh/projects/furniturestyle/
lang: zh-TW
translation_key: furniturestyle
categories: [Project]
tags: [Project, Web, AI Assisted]
published: true
---

[English version](/projects/furniturestyle/)

> I used AI to move quickly through this first batch of project articles. Each one was expanded from reports, code, or notes I already had, then checked against those sources before publishing.
>
> 這一批專案文章是我用 AI 快速補寫的。內容都從既有報告、程式碼或筆記展開，發布前再逐篇對回原始資料。

FurnitureStyle 是一個以 Django 開發的原型，讓使用者透過圖片或文字描述搜尋家具。主要流程會先把輸入轉成簡短的結構化描述，再用這段描述查詢購物搜尋結果。

這個原型可能是臺大 Web 應用程式設計課程的期末成果，但現有的課程資料沒有寫出專案名稱，也沒有留下最終評量。因此，這篇文章只描述目前可見的程式碼，不把它寫成已部署的產品或已確認的課程成果。

## 為什麼分類器要呼叫兩次模型

早期版本用一個多模態提示，同時辨識家具並改寫顏色描述。修訂後的流程把兩件事拆開：

```text
圖片或文字
-> 辨識家具類型、風格與原始主色
-> 依需求只轉換顏色描述
-> 用結構化的類型與風格搜尋商品
```

第一次模型回應必須是包含 `type` 與 `style` 的 JSON。如果使用者選擇保留原色，應用程式可以直接使用這份結果。互補色、單色與類似色模式則會觸發第二次請求，只修改顏色部分，保留家具本身的辨識結果。

這樣一來，應用程式可以看到中間的辨識內容，不必把辨識與轉換全塞進同一個提示。

## 分類器之外的應用程式

現有程式庫包含 Django 的 models、views、serializers、templates，以及購物搜尋整合。使用者資料會保存房間、顏色與風格偏好；收藏功能則讓使用者新增或移除商品。

![FurnitureStyle 的初步系統架構與搜尋流程](/assets/img/furniturestyle/system-flow.webp)
_期末資料中的初步流程圖。前端原始碼可對上 Vue 登入、搜尋、結果、收藏與歷史紀錄；OpenAI、購物搜尋、MySQL 與部署關係仍需後端材料才能完整驗證。_

現有的 Vue 3 前端讓使用流程更具體。使用者可以用帳號密碼或 Google 登入，輸入文字或上傳圖片，選擇原色、類似色或互補色模式，再進入結果頁。商品卡片可以連到商品頁並加入收藏，帳號頁也能查看搜尋紀錄。

我的實作範圍涵蓋使用者資料與家具資料模型的修改、收藏和個人資料功能，以及兩階段分類器與顏色模式處理。我與負責前端流程的成員協作，但現有前端原始碼中的作者身分仍未確認，因此我不把那部分列為自己的實作。

程式庫是私人資料，因此本文不提供連結，也不重製原始碼。

保留下來的期末專案封存資料包含報告、投影片、展示影片與上方的系統流程圖。另一份 RAR 則包含前端原始碼與建置後的 `dist` 目錄。配套的後端 ZIP 只有空目錄，無法進一步證明 Django 內部實作、部署或測試情況。

## 這份紀錄不主張什麼

目前沒有分類準確率、推薦品質、使用者採用或正式部署的評估紀錄。專案與課程的關係也是依時間與脈絡推定，並非課程提交頁面明確記載。

這個原型可以證明一條可運作的產品流程，以及一次具體的提示架構調整；但它不能證明搜尋結果已經適合支援真實的購買決策。
