# 머니플로우 (설치형 PWA + 두 폰 동기화)

월급·대출·생활비·투자 통합관리 + 미래 현금/순자산/부동산 차익 예측 모바일웹.
**Supabase**로 두 폰(내 폰 + 배우자 폰)이 **같은 데이터를 실시간 동기화**합니다.

## 파일 구성
- `index.html` — 앱 본체 (화면·계산·동기화 전부 포함)
- `manifest.webmanifest` — 홈화면 설치 정보
- `sw.js` — 오프라인 캐시(서비스워커)
- `icon-192.png` / `icon-512.png` / `icon-maskable.png` — 앱 아이콘

---

## 1단계 · Supabase 준비 (약 5분, 무료)

1. https://supabase.com 가입 → **New project** 생성 (Region은 `Northeast Asia (Seoul)` 권장).
2. 좌측 **SQL Editor** → 아래 SQL 붙여넣고 **Run**:

```sql
create table if not exists households (
  code text primary key,
  pass text,
  data jsonb not null default '{}'::jsonb,
  rev bigint not null default 0,
  updated_at timestamptz default now()
);

alter table households enable row level security;

-- 공유코드 방식: 추측 불가능한 가구코드 자체가 비밀키 역할.
create policy "hh_select" on households for select using (true);
create policy "hh_insert" on households for insert with check (true);
create policy "hh_update" on households for update using (true) with check (true);

-- 실시간 동기화 활성화
alter publication supabase_realtime add table households;
```

3. 좌측 **Project Settings → API** 에서 두 값을 복사해 둡니다:
   - **Project URL** (예: `https://abcd1234.supabase.co`)
   - **anon public** key (`eyJ...` 로 시작하는 긴 문자열) — 이건 공개돼도 안전한 키입니다.

> 앱 첫 화면에서 이 두 값을 입력하므로, 코드에 하드코딩할 필요 없습니다.

---

## 2단계 · 웹에 올리기 (GitHub Pages 추천 — 이미 사용 중)

1. GitHub에 새 저장소(예: `moneyflow`) 생성.
2. 이 폴더의 `index.html`, `manifest.webmanifest`, `sw.js`, `icon-*.png` 를 업로드/푸시.
3. 저장소 **Settings → Pages → Source: main / (root)** 선택 → 저장.
4. 몇 분 뒤 `https://<계정>.github.io/moneyflow/` 주소가 생깁니다.
   - ⚠️ PWA 설치와 서비스워커는 **HTTPS**에서만 동작합니다. GitHub Pages는 자동 HTTPS라 그대로 OK.

---

## 3단계 · 두 폰에 설치 + 연결

**내 폰**
1. 크롬(안드) / 사파리(아이폰)로 위 주소 접속.
2. 첫 화면에서 **Supabase URL + anon key** 입력.
3. **새 가구 만들기** 탭 → 자동 생성된 가구코드 확인 + **패스코드** 입력 → *가구 만들기*.
4. 브라우저 메뉴 → **홈 화면에 추가** (안드: 설치 배너 / 아이폰: 공유→홈 화면에 추가).

**배우자 폰**
1. 같은 주소 접속 → 같은 **URL + anon key** 입력.
2. **기존 가구 연결** 탭 → 내 폰에서 만든 **가구코드 + 패스코드** 입력 → *연결하고 불러오기*.
3. 홈 화면에 추가.

이제 한쪽에서 고치면 다른 폰에 몇 초 안에 반영됩니다. 상단 우측의 **동기화 뱃지**를 누르면 가구코드 확인·강제 동기화·연결 해제가 가능합니다.

---

## 동작 방식 메모
- 저장 단위: 앱 전체 데이터를 하나의 JSON(`data`)으로 저장, 변경 시 `rev` 증가.
- 충돌 처리: **마지막 저장 우선(last-write-wins)**. 두 사람이 동시에 같은 초에 편집하는 경우가 아니면 문제없음.
- 오프라인: 서비스워커가 앱을 캐시 → 인터넷 없이도 화면은 뜨고, 온라인 복귀 시 동기화.
- 로컬 백업: 각 기기 `localStorage`에도 항상 저장되므로 클라우드 장애 시에도 데이터 유지.

## 보안 참고 (공유코드 방식)
가구코드(24자 랜덤)가 사실상 비밀키입니다. 코드+패스코드를 아는 사람만 접근 가능.
더 강한 보안이 필요하면 나중에 '개인 계정 로그인(Supabase Auth)'으로 업그레이드할 수 있습니다.
