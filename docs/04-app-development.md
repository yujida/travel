# 04. 웹·앱 개발 조사

## 1. 결론부터

- **처음부터 네이티브 앱을 만들지 마세요.** 여행 상품은 1인당 구매 빈도가 낮아(1년에 1~2회) 앱을 설치하라고 하면 그 단계에서 고객이 빠져나갑니다. **판매는 모바일 웹**으로 합니다.
- 앱이 진짜 쓸모 있어지는 순간은 **여행 중**입니다. 일정, 실시간 공지, 작품 해설, 단체 소통이 필요할 때죠. 그래서 앱은 "여행 동반 앱(컴패니언)"으로 정의하고, 첫 출발에 맞춰 내놓습니다.
- **결제를 직접 받는 시점이 법·PG 문제의 갈림길입니다.** 여행업 등록 전(모델 A)에는 결제를 여행사에 맡기고, 우리 서비스는 "모객과 경험" 레이어만 맡습니다.

## 2. 단계별 로드맵

| 단계 | 시기 | 형태 | 핵심 기능 | 결제 |
|---|---|---|---|---|
| **P0 검증** | 즉시~1개월 | 노코드 랜딩 (아임웹, 노션 + Tally 폼 등) | 사전 알림 신청, 수요 설문, 카카오톡 채널 연결 | 없음 |
| **P1 판매 웹** | 2~3개월 | 자체 웹사이트 (반응형) | 상품 목록·상세, 대기·알림 신청, 남은 좌석 표시, 크리에이터 페이지, 관리자에서 상품 등록 | 여행사 결제 페이지로 이동 (모델 A) |
| **P2 여행 동반 앱** | 출발 1~2개월 전 | PWA 먼저 → 필요하면 앱스토어 앱 | 내 여행(일정·바우처·체크리스트), 공지·푸시, 작품 오디오 해설, 단체 채팅 또는 오픈채팅 연동, 사진 공유 앨범 | — |
| **P3 직접 판매** | 여행업 등록 후 | 웹 + 앱 | 예약·결제·분할결제(계약금/잔금), 마이페이지, 취소·환불 자동 계산 | 자체 PG 계약 |
| **P4 플랫폼** | 장기 | 크리에이터 어드민 | 크리에이터가 직접 상품을 기획·판매하고, 정산 대시보드 제공 (TrovaTrip형) | 정산 시스템 |

## 3. 기능 명세 (P1 + P2 중심)

### 고객용
- **상품**: 목록(행사·지역·월별 필터), 상세(03 문서 구성), 남은 좌석/매진/대기 상태
- **알림 신청 / 대기 신청**: 이름, 연락처, 관심 상품, 마케팅 수신 동의 → 카카오 알림톡이나 이메일로 오픈 알림
- **크리에이터 페이지**: 소개, 콘텐츠 임베드(유튜브), 이 크리에이터의 상품
- **내 여행 (P2)**: 일자별 일정, 집합 장소 지도, 준비물 체크리스트, 바우처·항공권(PDF), 긴급 연락처
- **공지·푸시 (P2)**: "내일 8시 로비 집합", 일정 변경
- **작품 노트 / 오디오 해설 (P2)**: 크리에이터가 녹음한 작품별 해설. 전시장에서는 그룹 해설이 어려운 경우가 많아 개인 이어폰으로 듣는 방식이 실용적임
- **사진 앨범 (P2)**: 단체 사진 공유, 귀국 후 아트북 제작 소스로 활용

### 운영자·크리에이터용 (관리자)
- 상품 CRUD (일정, 가격 구성, 포함/불포함, 이미지)
- 좌석 수와 상태 관리, 신청자·대기자 목록 내보내기 (여행사에 공유)
- 공지 작성과 푸시 발송
- 콘텐츠(오디오·작품 노트) 업로드

## 4. 기술 스택 제안

| 영역 | 추천 | 이유 |
|---|---|---|
| 웹 프론트 | **Next.js** (React, TypeScript) | 상세페이지 SEO·SSR, 생태계 |
| 백엔드·DB | **Supabase** (Postgres + Auth + Storage) | 인증·DB·파일 저장을 한 번에. 초기 인력이 적을 때 유리 |
| 호스팅 | Vercel | Next.js 배포가 가장 간단함 |
| 앱 | **PWA → Expo(React Native)** | 웹과 TypeScript 코드·인력을 공유. Flutter도 가능하지만 웹과 언어가 갈림 |
| 알림 | 카카오 알림톡 (발송 대행사), 이메일 (Resend 등), 앱 푸시 (Expo Notifications / FCM) | 한국 고객은 알림톡 도달률이 가장 높음 |
| 결제 (P3) | **PortOne(구 아임포트)** 연동 + 토스페이먼츠 등 PG | 여러 PG·간편결제를 한 SDK로. 계약금/잔금 분할 처리 |
| 분석 | GA4 + Meta 픽셀 | 유입 채널별 전환 측정 |
| CMS | 관리자 직접 구현 또는 Sanity 등 헤드리스 CMS | 상품·콘텐츠를 비개발자가 수정 |

## 5. 데이터 모델 초안

```
creators        (id, name, bio, profile_image, channels[])
products        (id, slug, title, subtitle, creator_id, status[draft|waitlist|on_sale|sold_out|closed],
                 depart_date, return_date, capacity, min_participants, confirm_deadline,
                 price, price_breakdown(json), includes(json), excludes(json),
                 agency_name, agency_reg_no, agency_booking_url, cancel_policy(md))
itinerary_days  (id, product_id, day_no, date, city, title, description, meals, hotel, map_points(json))
leads           (id, product_id, name, phone, email, interest(json), marketing_consent, created_at)
                -- 알림·대기 신청
bookings        (id, product_id, user_id, status, deposit_paid, balance_paid, ...)   -- P3
trip_members    (id, product_id, user_id, room_type, roommate_id)                    -- P2
announcements   (id, product_id, title, body, sent_at)                               -- P2
artwork_notes   (id, product_id, day_no, venue, artist, title, text, audio_url, image_url) -- P2
```

## 6. 결제·법규 리스크 (개발에 영향 주는 것)

- **PG 심사:** PortOne 헬프센터는 여행(교통·숙박·투어) 플랫폼을 **고위험 업종**으로 분류합니다. 여행업 등록은 필수이고, 업력과 신용등급을 중점적으로 심사합니다. 초기 법인이 직접 PG를 받기는 어렵다고 봐야 합니다. → P1에서는 여행사 결제를 쓰는 것이 현실적입니다.
- **정산 구조:** 플랫폼이 결제대금을 직접 받아 다른 판매자에게 정산하면 전자지급결제대행업(PG업) 등록 이슈가 생길 수 있습니다. 정산을 외부 PG사가 대행하면 등록이 필요하지 않다는 것이 금융위원회 설명입니다. P4 플랫폼 단계에서는 PG사의 정산 대행(오픈마켓형) 상품을 쓰세요.
- **통신판매업 신고**와 **이용약관·개인정보 처리방침·환불 규정 페이지**는 PG 심사 서류에 해당합니다. P1부터 페이지를 만들어 두세요.
- **개인정보:** 여권번호 같은 고유식별정보는 가능하면 우리 서버에 저장하지 말고 여행사 시스템으로 바로 넘기세요. 저장해야 한다면 암호화하고 별도 동의를 받아야 합니다.

## 7. 개발 규모 감 잡기 (1인 풀스택 기준 · 대략치)

| 단계 | 작업량 | 비고 |
|---|---|---|
| P0 노코드 랜딩 | 2~3일 | 개발자 없이 가능 |
| P1 판매 웹 (상품·상세·알림 신청·관리자) | 4~6주 | 디자인 별도 |
| P2 PWA 동반 앱 (일정·공지·오디오·앨범) | 4~6주 | 앱스토어 출시는 +2~3주 (심사 포함) |
| P3 자체 결제·예약 | 4~8주 | PG 계약·심사 기간 별도 |

외주를 쓴다면 위 범위를 RFP(제안요청서)로 쪼개서 견적을 받는 것을 추천합니다. 이 레포에서 바로 P1을 개발할 수도 있습니다.

## 8. 다음 액션

1. P0 랜딩에 넣을 문구와 폼 항목 확정 (03 문서 6절 v0)
2. 브랜드명·도메인 결정
3. 여행사 협업 구조가 정해지면 P1 개발 시작 (Next.js + Supabase 프로젝트 세팅)

## 출처
- [PortOne 고위험 업종 안내](https://help.portone.io/content/high-risk-industry)
- [금융위원회 보도설명: 결제대금 정산과 전자지급결제대행업](https://www.fsc.go.kr/no010102/82523)
- [PG 심사 기준 (페이업 블로그)](https://blog.payup.co.kr/pg-approval-without-website)
- [해외결제 PG 비교 (PortOne 블로그)](https://blog.portone.io/opi_pg-comparison_global/)
- [TrovaTrip Host](https://trovatrip.com/host)
