---
layout: post
title: Deep Learning CNN
date: 2026-11-04 00:00 +0800
last_modified_at: 2025-04-10 00:08:25 +0800
tags: [AI, Deep Learnig, GAN ]
categoie: [AI]
topic:  true

---
**GAN(Generative adversarial network)**!
{: .message }

## 개요
GAN은 두 개의 신경망이 서로 경쟁하면서 학습하는 구조입니다.

생성자(Generator, G)
 -  진짜처럼 보이는 데이터를 생성해내는 모델  
예: 노이즈 → 가짜 고양이 사진 생성

판별자(Discriminator, D)
- 진짜 데이터와 가짜 데이터를 구분하는 모델  
예: 진짜 고양이 사진 vs. G가 만든 가짜 고양이 사진을 구분

둘은 경쟁 게임을 해:

G는 더 진짜같이 속이려고 하고  
D는 더 잘 구분하려고 학습해 결국 G가 너무 잘 만들어서 D도 헷갈리면, 학습 성공

