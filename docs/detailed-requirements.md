# 세부 요구 사항 문서: MemoryMatch+ MVP

**문서 버전**: 1.0  
**작성일**: 2025-01-27  
**기준 문서**: PRD v1.0  
**범위**: Phase 1 (MVP)

---

## 목차

1. [개요](#1-개요)
2. [기능 상세 명세](#2-기능-상세-명세)
3. [UI/UX 상세 설계](#3-uiux-상세-설계)
4. [데이터 모델](#4-데이터-모델)
5. [컴포넌트 설계](#5-컴포넌트-설계)
6. [게임 로직 명세](#6-게임-로직-명세)
7. [에러 처리 및 엣지 케이스](#7-에러-처리-및-엣지-케이스)
8. [성능 및 최적화 요구사항](#8-성능-및-최적화-요구사항)
9. [테스트 요구사항](#9-테스트-요구사항)

---

## 1. 개요

### 1.1 문서 목적
이 문서는 MemoryMatch+ MVP 개발을 위한 상세 기술 명세서입니다. 개발자가 구현에 필요한 모든 세부사항을 포함합니다.

### 1.2 MVP 범위
- 클래식 모드 (Easy, Medium 난이도)
- 기본 테마 2종 (동물, 과일)
- 점수 및 시간 기록 시스템
- LocalStorage 기반 데이터 저장
- 반응형 UI (모바일/데스크톱)

### 1.3 기술 스택
- **프레임워크**: React 18+
- **상태 관리**: React Context API 또는 Zustand
- **스타일링**: Tailwind CSS
- **애니메이션**: CSS Transitions + Framer Motion (선택)
- **빌드 도구**: Vite 또는 Create React App

---

## 2. 기능 상세 명세

### 2.1 클래식 모드

#### 2.1.1 게임 규칙
1. 모든 카드는 뒷면이 보이도록 배치됨
2. 사용자가 카드를 클릭하면 앞면이 보임
3. 두 번째 카드를 클릭하면 두 카드 비교
4. 일치하면 카드 제거, 불일치하면 다시 뒤집힘
5. 모든 카드 쌍을 찾으면 게임 종료

#### 2.1.2 난이도별 설정

**Easy 모드**
- 그리드 크기: 4x3 (12장, 6쌍)
- 카드 크기: 데스크톱 120px × 120px, 모바일 80px × 80px
- 최대 시도 횟수: 제한 없음
- 시간 제한: 없음

**Medium 모드**
- 그리드 크기: 4x4 (16장, 8쌍)
- 카드 크기: 데스크톱 100px × 100px, 모바일 70px × 70px
- 최대 시도 횟수: 제한 없음
- 시간 제한: 없음

#### 2.1.3 카드 상태
- `hidden`: 뒷면 (기본 상태)
- `revealed`: 앞면 (클릭 후)
- `matched`: 매칭 완료 (제거됨)
- `mismatched`: 불일치 (일시적으로 표시)

### 2.2 점수 시스템

#### 2.2.1 점수 계산 규칙
```
기본 점수 = 100점 (매칭 성공 시)
콤보 보너스 = (연속 매칭 횟수 - 1) × 10점
최종 점수 = 기본 점수 + 콤보 보너스
```

**콤보 시스템**
- 첫 매칭: 100점
- 2연속 매칭: 110점 (100 + 10)
- 3연속 매칭: 120점 (100 + 20)
- 4연속 매칭: 130점 (100 + 30)
- 불일치 발생 시 콤보 리셋

#### 2.2.2 실수 페널티
- 불일치 시: -5점
- 점수는 0점 이하로 내려가지 않음

### 2.3 시간 기록

#### 2.3.1 시간 추적
- 게임 시작 시 타이머 시작
- 게임 종료 시 타이머 정지
- 형식: MM:SS (예: 02:35)
- 최소 단위: 1초

#### 2.3.2 일시정지 기능
- 일시정지 버튼 클릭 시 타이머 정지
- 카드 클릭 비활성화
- 일시정지 모달 표시
- 재개 시 타이머 계속 진행

### 2.4 통계 추적

#### 2.4.1 게임별 기록
- 완료 시간 (초 단위)
- 시도 횟수 (카드 클릭 쌍 수)
- 정확도 (성공 매칭 / 전체 시도 × 100)
- 획득 점수
- 난이도
- 테마
- 플레이 날짜/시간

#### 2.4.2 누적 통계
- 총 플레이 시간 (초)
- 총 게임 수
- 평균 완료 시간
- 최고 점수
- 난이도별 최고 기록

### 2.5 테마 시스템

#### 2.5.1 기본 테마

**동물 테마**
- 카드 쌍: 강아지, 고양이, 토끼, 펭귄, 곰, 사자
- 색상 팔레트: 따뜻한 톤
- 이미지 형식: SVG 또는 PNG (최소 200x200px)

**과일 테마**
- 카드 쌍: 사과, 바나나, 딸기, 오렌지, 포도, 수박
- 색상 팔레트: 밝고 생동감 있는 색상
- 이미지 형식: SVG 또는 PNG (최소 200x200px)

#### 2.5.2 테마 선택
- 메인 화면에서 드롭다운으로 선택
- 선택한 테마는 LocalStorage에 저장
- 다음 게임 시작 시 기본값으로 적용

---

## 3. UI/UX 상세 설계

### 3.1 화면 구성

#### 3.1.1 메인 화면 (HomeScreen)

**레이아웃 구조**
```
┌─────────────────────────────────┐
│        MemoryMatch+ 로고         │
├─────────────────────────────────┤
│  [게임 시작] 버튼 (Primary)      │
├─────────────────────────────────┤
│  모드: [클래식] (고정)           │
│  난이도: [Easy] [Medium]        │
│  테마: [드롭다운: 동물 ▼]        │
├─────────────────────────────────┤
│  [내 기록 보기] 버튼 (Secondary) │
│  [설정] 버튼 (Tertiary)          │
└─────────────────────────────────┘
```

**컴포넌트 구성**
- `Header`: 로고 및 타이틀
- `GameStartButton`: 메인 CTA 버튼
- `ModeSelector`: 모드 선택 (MVP에서는 "클래식" 고정)
- `DifficultySelector`: 난이도 선택 버튼 그룹
- `ThemeSelector`: 테마 선택 드롭다운
- `NavigationButtons`: 기록 보기, 설정 버튼

**상태 관리**
- 선택된 난이도: `easy` | `medium`
- 선택된 테마: `animals` | `fruits`
- 버튼 활성화 상태

#### 3.1.2 게임 화면 (GameScreen)

**레이아웃 구조**
```
┌─────────────────────────────────┐
│ [일시정지] 점수: 500 시도: 5     │
│ 시간: 01:23                      │
├─────────────────────────────────┤
│                                 │
│    [카드 그리드 4x3 또는 4x4]    │
│                                 │
├─────────────────────────────────┤
│        [힌트] (선택사항)         │
└─────────────────────────────────┘
```

**커포넌트 구성**
- `GameHeader`: 점수, 시간, 시도 횟수, 일시정지 버튼
- `CardGrid`: 카드 그리드 컨테이너
- `Card`: 개별 카드 컴포넌트
- `HintButton`: 힌트 버튼 (MVP에서는 선택사항)

**반응형 레이아웃**
- 데스크톱: 그리드 중앙 정렬, 여백 충분
- 태블릿: 그리드 크기 조정
- 모바일: 그리드 크기 축소, 터치 최적화

#### 3.1.3 결과 화면 (ResultScreen)

**레이아웃 구조**
```
┌─────────────────────────────────┐
│      🎉 게임 완료! 🎉           │
├─────────────────────────────────┤
│  최종 점수: 850점               │
│  완료 시간: 02:35               │
│  시도 횟수: 12회                │
│  정확도: 75%                    │
├─────────────────────────────────┤
│  [다시 하기] [메인으로]         │
└─────────────────────────────────┘
```

**컴포넌트 구성**
- `ResultHeader`: 축하 메시지
- `ResultStats`: 통계 표시
- `ResultActions`: 액션 버튼 그룹

### 3.2 인터랙션 디자인

#### 3.2.1 카드 애니메이션

**카드 뒤집기**
- 트랜지션 시간: 300ms
- 효과: 3D 회전 (rotateY 0deg → 180deg)
- Easing: ease-in-out
- 클릭 시 즉시 시작

**매칭 성공**
- 페이드아웃: 500ms
- 스케일 축소: scale(1) → scale(0.8)
- 완료 후 그리드에서 제거

**매칭 실패**
- 500ms 대기 후 뒤집기
- 회전 애니메이션 (180deg → 0deg)
- 300ms 트랜지션

#### 3.2.2 피드백

**시각적 피드백**
- 카드 호버: 약간의 그림자 증가
- 카드 클릭: 약간의 스케일 다운 (0.95)
- 콤보 표시: 화면 상단에 "2 COMBO!" 텍스트 (2초간 표시)
- 점수 증가: 점수 숫자에 펄스 애니메이션

**사운드 피드백 (선택사항)**
- 카드 뒤집기: 부드러운 "flip" 사운드
- 매칭 성공: 긍정적인 "success" 사운드
- 매칭 실패: 중립적인 "fail" 사운드
- 게임 완료: 축하 "complete" 사운드

### 3.3 반응형 디자인

#### 3.3.1 브레이크포인트
```css
Mobile: < 768px
Tablet: 768px - 1024px
Desktop: > 1024px
```

#### 3.3.2 레이아웃 조정

**모바일 (< 768px)**
- 카드 크기: 70-80px
- 그리드 간격: 8px
- 버튼 크기: 최소 44px × 44px (터치 최적화)
- 폰트 크기: 14-16px
- 헤더 높이: 60px

**태블릿 (768px - 1024px)**
- 카드 크기: 90-100px
- 그리드 간격: 12px
- 버튼 크기: 48px × 48px
- 폰트 크기: 16-18px

**데스크톱 (> 1024px)**
- 카드 크기: 100-120px
- 그리드 간격: 16px
- 버튼 크기: 52px × 52px
- 폰트 크기: 18-20px

### 3.4 접근성

#### 3.4.1 키보드 네비게이션
- Tab: 버튼 간 이동
- Enter/Space: 버튼 활성화, 카드 선택
- Escape: 일시정지 모달 닫기, 게임 중단

#### 3.4.2 ARIA 레이블
- 모든 인터랙티브 요소에 적절한 aria-label
- 카드에 aria-label="카드 {번호}, {테마명}"
- 게임 상태를 screen reader에 알림

#### 3.4.3 색상 대비
- WCAG AA 기준 준수 (최소 4.5:1)
- 색상만으로 정보 전달하지 않음 (아이콘/텍스트 병행)

---

## 4. 데이터 모델

### 4.1 게임 상태 (Game State)

```typescript
interface GameState {
  // 게임 설정
  mode: 'classic';
  difficulty: 'easy' | 'medium';
  theme: 'animals' | 'fruits';
  
  // 게임 진행 상태
  status: 'idle' | 'playing' | 'paused' | 'finished';
  cards: Card[];
  flippedCards: number[]; // 현재 뒤집힌 카드 인덱스 (최대 2개)
  matchedPairs: number[]; // 매칭된 카드 쌍 인덱스
  attempts: number; // 시도 횟수
  score: number;
  combo: number; // 현재 콤보 수
  startTime: number | null; // 게임 시작 시간 (timestamp)
  elapsedTime: number; // 경과 시간 (초)
  pauseStartTime: number | null; // 일시정지 시작 시간
  totalPauseTime: number; // 총 일시정지 시간 (초)
}
```

### 4.2 카드 데이터 구조

```typescript
interface Card {
  id: number; // 고유 ID (0부터 시작)
  pairId: number; // 같은 쌍의 카드는 같은 pairId
  theme: 'animals' | 'fruits';
  image: string; // 이미지 URL 또는 경로
  state: 'hidden' | 'revealed' | 'matched' | 'mismatched';
}
```

### 4.3 게임 기록 (Game Record)

```typescript
interface GameRecord {
  id: string; // 고유 ID (UUID 또는 timestamp)
  mode: 'classic';
  difficulty: 'easy' | 'medium';
  theme: 'animals' | 'fruits';
  score: number;
  elapsedTime: number; // 초 단위
  attempts: number;
  accuracy: number; // 0-100
  playedAt: number; // timestamp
}
```

### 4.4 통계 데이터 (Statistics)

```typescript
interface Statistics {
  totalGames: number;
  totalPlayTime: number; // 초 단위
  averageTime: number; // 초 단위
  bestScore: number;
  bestTime: {
    easy: number | null; // 초 단위
    medium: number | null;
  };
  difficultyStats: {
    easy: {
      gamesPlayed: number;
      averageTime: number;
      bestScore: number;
    };
    medium: {
      gamesPlayed: number;
      averageTime: number;
      bestScore: number;
    };
  };
}
```

### 4.5 사용자 설정 (User Settings)

```typescript
interface UserSettings {
  theme: 'animals' | 'fruits';
  difficulty: 'easy' | 'medium';
  soundEnabled: boolean;
  animationsEnabled: boolean;
  reducedMotion: boolean; // 접근성 옵션
}
```

### 4.6 LocalStorage 스키마

```typescript
// 키 구조
const STORAGE_KEYS = {
  GAME_RECORDS: 'memorymatch_game_records', // GameRecord[]
  STATISTICS: 'memorymatch_statistics', // Statistics
  SETTINGS: 'memorymatch_settings', // UserSettings
};
```

**데이터 저장 형식**
- JSON 문자열로 저장
- 배열은 최대 100개까지만 저장 (오래된 기록 삭제)
- 데이터 유효성 검증 필요

---

## 5. 컴포넌트 설계

### 5.1 컴포넌트 계층 구조

```
App
├── Router (또는 상태 기반 화면 전환)
│   ├── HomeScreen
│   │   ├── Header
│   │   ├── GameStartButton
│   │   ├── ModeSelector
│   │   ├── DifficultySelector
│   │   │   └── DifficultyButton
│   │   ├── ThemeSelector
│   │   └── NavigationButtons
│   │       ├── RecordsButton
│   │       └── SettingsButton
│   │
│   ├── GameScreen
│   │   ├── GameHeader
│   │   │   ├── ScoreDisplay
│   │   │   ├── TimerDisplay
│   │   │   ├── AttemptsDisplay
│   │   │   └── PauseButton
│   │   ├── CardGrid
│   │   │   └── Card (×12 or ×16)
│   │   └── HintButton (선택사항)
│   │
│   ├── ResultScreen
│   │   ├── ResultHeader
│   │   ├── ResultStats
│   │   └── ResultActions
│   │       ├── PlayAgainButton
│   │       └── HomeButton
│   │
│   ├── RecordsScreen
│   │   ├── RecordsHeader
│   │   ├── StatisticsPanel
│   │   └── RecordsList
│   │       └── RecordItem
│   │
│   └── SettingsScreen
│       ├── SettingsHeader
│       └── SettingsForm
│
└── Modal
    ├── PauseModal
    └── ConfirmModal
```

### 5.2 주요 컴포넌트 명세

#### 5.2.1 Card 컴포넌트

```typescript
interface CardProps {
  card: Card;
  index: number;
  onClick: (index: number) => void;
  disabled: boolean; // 다른 카드가 뒤집혀 있을 때
}

// Props
- card: Card 객체
- index: 카드 인덱스
- onClick: 클릭 핸들러
- disabled: 클릭 비활성화 여부

// State
- 없음 (부모에서 관리)

// 동작
- 클릭 시 onClick(index) 호출
- state에 따라 다른 스타일 적용
- 애니메이션 트리거
```

#### 5.2.2 CardGrid 컴포넌트

```typescript
interface CardGridProps {
  cards: Card[];
  difficulty: 'easy' | 'medium';
  onCardClick: (index: number) => void;
  disabled: boolean;
}

// Props
- cards: Card 배열
- difficulty: 난이도 (그리드 크기 결정)
- onCardClick: 카드 클릭 핸들러
- disabled: 전체 그리드 비활성화 (일시정지 등)

// 동작
- difficulty에 따라 그리드 레이아웃 결정
- CSS Grid로 카드 배치
- 반응형 크기 조정
```

#### 5.2.3 GameHeader 컴포넌트

```typescript
interface GameHeaderProps {
  score: number;
  elapsedTime: number;
  attempts: number;
  onPause: () => void;
}

// Props
- score: 현재 점수
- elapsedTime: 경과 시간 (초)
- attempts: 시도 횟수
- onPause: 일시정지 핸들러

// 동작
- 1초마다 타이머 업데이트
- 점수 포맷팅 (천 단위 구분)
- 시도 횟수 표시
```

#### 5.2.4 DifficultySelector 컴포넌트

```typescript
interface DifficultySelectorProps {
  selected: 'easy' | 'medium';
  onChange: (difficulty: 'easy' | 'medium') => void;
}

// Props
- selected: 선택된 난이도
- onChange: 난이도 변경 핸들러

// 동작
- 버튼 그룹으로 난이도 선택
- 선택된 버튼 하이라이트
```

### 5.3 상태 관리 구조

#### 5.3.1 Context API 구조

```typescript
// GameContext
interface GameContextValue {
  gameState: GameState;
  startGame: (config: GameConfig) => void;
  flipCard: (index: number) => void;
  pauseGame: () => void;
  resumeGame: () => void;
  resetGame: () => void;
}

// SettingsContext
interface SettingsContextValue {
  settings: UserSettings;
  updateSettings: (updates: Partial<UserSettings>) => void;
}

// RecordsContext
interface RecordsContextValue {
  records: GameRecord[];
  statistics: Statistics;
  addRecord: (record: GameRecord) => void;
  clearRecords: () => void;
}
```

#### 5.3.2 상태 업데이트 플로우

```
사용자 액션
  ↓
이벤트 핸들러
  ↓
Context Action
  ↓
State 업데이트
  ↓
컴포넌트 리렌더링
```

---

## 6. 게임 로직 명세

### 6.1 게임 초기화

#### 6.1.1 카드 생성 알고리즘

```typescript
function createCards(difficulty: 'easy' | 'medium', theme: 'animals' | 'fruits'): Card[] {
  // 1. 난이도에 따른 쌍 수 결정
  const pairCount = difficulty === 'easy' ? 6 : 8;
  
  // 2. 테마에 따른 이미지 배열 가져오기
  const images = getThemeImages(theme, pairCount);
  
  // 3. 카드 쌍 생성
  const pairs: Card[] = [];
  for (let i = 0; i < pairCount; i++) {
    const pairId = i;
    // 각 쌍당 2장 생성
    pairs.push({
      id: i * 2,
      pairId,
      theme,
      image: images[i],
      state: 'hidden'
    });
    pairs.push({
      id: i * 2 + 1,
      pairId,
      theme,
      image: images[i],
      state: 'hidden'
    });
  }
  
  // 4. 셔플
  return shuffleArray(pairs);
}
```

#### 6.1.2 셔플 알고리즘 (Fisher-Yates)

```typescript
function shuffleArray<T>(array: T[]): T[] {
  const shuffled = [...array];
  for (let i = shuffled.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
  }
  return shuffled;
}
```

### 6.2 카드 뒤집기 로직

#### 6.2.1 플로우차트

```
카드 클릭
  ↓
이미 뒤집힌 카드인가? → Yes → 무시
  ↓ No
이미 2장이 뒤집혀 있는가? → Yes → 무시
  ↓ No
matched 상태인가? → Yes → 무시
  ↓ No
카드를 revealed 상태로 변경
  ↓
flippedCards에 추가
  ↓
2장이 뒤집혔는가?
  ↓ Yes
매칭 검증
  ↓
일치 → matched 처리, 점수 추가, 콤보 증가
  ↓
불일치 → mismatched 처리, 페널티, 콤보 리셋
  ↓
500ms 후 다시 hidden 상태로
```

#### 6.2.2 매칭 검증

```typescript
function checkMatch(card1: Card, card2: Card): boolean {
  return card1.pairId === card2.pairId;
}
```

### 6.3 점수 계산 로직

#### 6.3.1 매칭 성공 시

```typescript
function calculateMatchScore(combo: number): number {
  const baseScore = 100;
  const comboBonus = (combo - 1) * 10;
  return baseScore + comboBonus;
}

// 예시
// 첫 매칭 (combo = 1): 100 + 0 = 100점
// 2연속 (combo = 2): 100 + 10 = 110점
// 3연속 (combo = 3): 100 + 20 = 120점
```

#### 6.3.2 매칭 실패 시

```typescript
function applyMismatchPenalty(currentScore: number): number {
  const penalty = 5;
  return Math.max(0, currentScore - penalty); // 최소 0점
}
```

### 6.4 게임 종료 조건

```typescript
function isGameComplete(cards: Card[]): boolean {
  return cards.every(card => card.state === 'matched');
}
```

### 6.5 시간 추적 로직

#### 6.5.1 타이머 구현

```typescript
// 게임 시작
startTime = Date.now();
pauseStartTime = null;
totalPauseTime = 0;

// 일시정지
pauseStartTime = Date.now();

// 재개
if (pauseStartTime) {
  totalPauseTime += Date.now() - pauseStartTime;
  pauseStartTime = null;
}

// 경과 시간 계산
elapsedTime = Math.floor((Date.now() - startTime - totalPauseTime) / 1000);
```

#### 6.5.2 시간 포맷팅

```typescript
function formatTime(seconds: number): string {
  const mins = Math.floor(seconds / 60);
  const secs = seconds % 60;
  return `${String(mins).padStart(2, '0')}:${String(secs).padStart(2, '0')}`;
}
```

### 6.6 정확도 계산

```typescript
function calculateAccuracy(matchedPairs: number, attempts: number): number {
  if (attempts === 0) return 0;
  return Math.round((matchedPairs / attempts) * 100);
}
```

---

## 7. 에러 처리 및 엣지 케이스

### 7.1 예외 상황

#### 7.1.1 LocalStorage 오류
- **상황**: LocalStorage가 가득 찬 경우, 비활성화된 경우
- **처리**: 
  - try-catch로 감싸기
  - 오류 발생 시 콘솔 경고 및 메모리에서만 동작
  - 사용자에게 알림 표시 (선택사항)

#### 7.1.2 이미지 로딩 실패
- **상황**: 카드 이미지가 로드되지 않는 경우
- **처리**:
  - 기본 플레이스홀더 이미지 표시
  - alt 텍스트로 대체
  - 에러 이벤트 핸들러로 fallback 이미지 로드

#### 7.1.3 게임 상태 불일치
- **상황**: 예상치 못한 상태 전이
- **처리**:
  - 상태 검증 로직 추가
  - 잘못된 상태 감지 시 게임 리셋 옵션 제공

### 7.2 엣지 케이스

#### 7.2.1 빠른 연속 클릭
- **문제**: 사용자가 매우 빠르게 카드를 클릭
- **처리**: 
  - debounce 또는 disabled 플래그로 방지
  - 2장이 뒤집혀 있을 때 추가 클릭 무시

#### 7.2.2 같은 카드 중복 클릭
- **문제**: 이미 뒤집힌 카드를 다시 클릭
- **처리**: 
  - revealed 상태인 카드 클릭 무시
  - matched 상태인 카드 클릭 무시

#### 7.2.3 브라우저 탭 전환
- **문제**: 다른 탭으로 이동 후 돌아올 때
- **처리**:
  - Page Visibility API로 감지
  - 탭이 비활성화되면 자동 일시정지 (선택사항)
  - 또는 계속 진행 (기본 동작)

#### 7.2.4 화면 크기 변경
- **문제**: 게임 중 브라우저 크기 조정
- **처리**:
  - ResizeObserver 또는 window resize 이벤트
  - 레이아웃 자동 재조정
  - 게임 상태 유지

#### 7.2.5 네트워크 오프라인
- **문제**: 이미지가 CDN에서 로드되는 경우
- **처리**:
  - 오프라인 감지
  - 로컬 이미지 사용 또는 캐시 활용

### 7.3 데이터 무결성

#### 7.3.1 LocalStorage 데이터 검증
```typescript
function validateStorageData(key: string, expectedType: string): boolean {
  try {
    const data = localStorage.getItem(key);
    if (!data) return false;
    const parsed = JSON.parse(data);
    // 타입 검증 로직
    return typeof parsed === expectedType;
  } catch {
    return false;
  }
}
```

#### 7.3.2 데이터 마이그레이션
- 버전 관리: 스키마 변경 시 마이그레이션 로직
- 호환성: 이전 버전 데이터 처리

---

## 8. 성능 및 최적화 요구사항

### 8.1 렌더링 최적화

#### 8.1.1 React 최적화
- `React.memo`로 Card 컴포넌트 메모이제이션
- `useMemo`로 카드 배열 메모이제이션
- `useCallback`으로 이벤트 핸들러 메모이제이션
- 불필요한 리렌더링 방지

#### 8.1.2 이미지 최적화
- 이미지 lazy loading
- WebP 형식 지원 (fallback PNG)
- 적절한 이미지 크기 (200x200px 이상, 불필요하게 크지 않게)
- 이미지 스프라이트 또는 SVG 사용 고려

### 8.2 메모리 관리

#### 8.2.1 상태 관리
- 불필요한 상태 저장 방지
- 큰 객체의 깊은 복사 최소화
- 게임 종료 시 불필요한 리소스 정리

#### 8.2.2 이벤트 리스너
- 컴포넌트 언마운트 시 이벤트 리스너 제거
- 타이머 정리 (setInterval, setTimeout)

### 8.3 애니메이션 성능

#### 8.3.1 CSS 최적화
- `transform`과 `opacity`만 애니메이션 (GPU 가속)
- `will-change` 속성 적절히 사용
- `requestAnimationFrame` 활용

#### 8.3.2 애니메이션 감소 옵션
- 사용자 설정에서 애니메이션 비활성화 옵션
- `prefers-reduced-motion` 미디어 쿼리 지원

### 8.4 로딩 성능

#### 8.4.1 초기 로딩
- 코드 스플리팅 (React.lazy)
- 이미지 프리로딩
- 최소한의 초기 번들 크기

#### 8.4.2 지연 로딩
- 결과 화면, 기록 화면은 필요 시 로드
- 테마 이미지는 선택된 테마만 로드

### 8.5 성능 목표

- 초기 로딩: 3초 이내
- 카드 클릭 반응: 100ms 이내
- 애니메이션: 60fps 유지
- 메모리 사용: 50MB 이하 (모바일 기준)

---

## 9. 테스트 요구사항

### 9.1 단위 테스트

#### 9.1.1 게임 로직 테스트
- 카드 생성 및 셔플
- 매칭 검증
- 점수 계산
- 정확도 계산
- 게임 종료 조건

#### 9.1.2 유틸리티 함수 테스트
- 시간 포맷팅
- LocalStorage 읽기/쓰기
- 데이터 검증

### 9.2 통합 테스트

#### 9.2.1 게임 플로우 테스트
- 게임 시작 → 진행 → 종료 전체 플로우
- 일시정지 및 재개
- 기록 저장 및 불러오기

#### 9.2.2 상태 관리 테스트
- Context 업데이트
- 상태 전이
- 데이터 동기화

### 9.3 E2E 테스트 (선택사항)

#### 9.3.1 사용자 시나리오
- 메인 화면에서 게임 시작
- 게임 완료까지 플레이
- 기록 확인
- 설정 변경

### 9.4 접근성 테스트

#### 9.4.1 키보드 네비게이션
- 모든 기능이 키보드로 접근 가능한지
- 포커스 관리가 적절한지

#### 9.4.2 Screen Reader
- ARIA 레이블이 적절한지
- 게임 상태가 올바르게 전달되는지

### 9.5 크로스 브라우저 테스트

#### 9.5.1 지원 브라우저
- Chrome (최신 2개 버전)
- Firefox (최신 2개 버전)
- Safari (최신 2개 버전)
- Edge (최신 2개 버전)

#### 9.5.2 모바일 브라우저
- iOS Safari
- Chrome Mobile
- Samsung Internet

### 9.6 성능 테스트

#### 9.6.1 Lighthouse 점수
- Performance: 90 이상
- Accessibility: 90 이상
- Best Practices: 90 이상
- SEO: N/A (게임 앱)

#### 9.6.2 실제 성능 측정
- 초기 로딩 시간
- 카드 클릭 반응 시간
- 애니메이션 프레임레이트

---

## 부록

### A. 용어 정의

- **매칭**: 같은 pairId를 가진 두 카드를 찾는 행위
- **시도**: 두 카드를 뒤집어 비교하는 행위
- **콤보**: 연속으로 성공한 매칭 횟수
- **정확도**: 성공한 매칭 / 전체 시도 × 100

### B. 참고 자료

- PRD 문서: `docs/prd.md`
- 브레인스토밍 문서: `docs/brainstorming.md`
- React 공식 문서: https://react.dev
- Tailwind CSS 문서: https://tailwindcss.com

### C. 변경 이력

| 버전 | 날짜 | 변경 내용 | 작성자 |
|------|------|----------|--------|
| 1.0 | 2025-01-27 | 초기 문서 작성 | - |

---

**문서 끝**

