# 📚 가상 면접 사례로 배우는 대규모 시스템 설계 기초

> **알렉스 쉬(Alex Xu) 저 · 이병준 역 · 인사이트**

---

## 🎯 스터디 소개

대규모 트래픽을 감당하는 시스템을 어떻게 설계하는지, 16가지 실제 면접 사례를 통해 함께 학습하고 토론하는 모임입니다.
요구사항 정의 → 개략적 설계 → 상세 설계 → 병목 개선으로 이어지는 설계 사고의 흐름을 익히는 것을 목표로 합니다.

---

## 👥 스터디원
<table>
  <tr>
    <td align="center">
      <a href="https://github.com/incheol789">
        <img src="https://github.com/incheol789.png" width="120" /><br/>
        <b>정인철</b>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/da4isy">
        <img src="https://github.com/da4isy.png" width="120" /><br/>
        <b>정다희</b>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/anewjean">
        <img src="https://github.com/anewjean.png" width="120" /><br/>
        <b>안유진</b>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/dd-jiny">
        <img src="https://github.com/dd-jiny.png" width="120" /><br/>
        <b>김대진</b>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/lim-jaein">
        <img src="https://github.com/lim-jaein.png" width="120" /><br/>
        <b>임재인</b>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/iohyeon">
        <img src="https://github.com/iohyeon.png" width="120" /><br/>
        <b>임나현</b>
      </a>
    </td>
  </tr>
  <tr>
    <td align="center">
      <a href="https://github.com/zeexzeex">
        <img src="https://github.com/zeexzeex.png" width="120" /><br/>
        <b>김지혜</b>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/yewonahn">
        <img src="https://github.com/yewonahn.png" width="120" /><br/>
        <b>안예원</b>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/essaysir">
        <img src="https://github.com/essaysir.png" width="120" /><br/>
        <b>손주선</b>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/madirony">
        <img src="https://github.com/madirony.png" width="120" /><br/>
        <b>연정흠</b>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/shAn-kor">
        <img src="https://github.com/shAn-kor.png" width="120" /><br/>
        <b>안성훈</b>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/yoon-yoo-tak">
        <img src="https://github.com/yoon-yoo-tak.png" width="120" /><br/>
        <b>윤유탁</b>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/Namjin-kimm">
        <img src="https://github.com/Namjin-kimm.png" width="120" /><br/>
        <b>김남진</b>
      </a>
    </td>
  </tr>
</table>

---

## 📅 진행 방식

| 항목 | 내용 |
|------|------|
| **일시** | 매주 토요일 오전 10시 |
| **분량** | 2주당 1개 챕터 |
| **방식** | 각자 해당 챕터를 **어떻게 구현·설계할지 미리 짜와서** 공유하고 비교 |
| **조 편성** | 챕터마다 조를 **랜덤 배정** |
| **자료** | 각자(또는 조별) 설계안을 PR로 제출 → 리뷰 후 머지 |


---

## 📖 챕터 구성

| 주차 | 챕터 | 주제 | 발표자 |
|:---:|:---:|------|:---:|
| 1 | Ch.01 | 사용자 수에 따른 규모 확장성 | |
| 1 | Ch.02 | 개략적인 규모 추정 | |
| 1 | Ch.03 | 시스템 설계 면접 공략법 | |
| 2 | Ch.04 | 처리율 제한 장치의 설계 | |
| 2 | Ch.05 | 안정 해시 설계 | |
| 3 | Ch.06 | 키-값 저장소 설계 | |
| 3 | Ch.07 | 분산 시스템을 위한 유일 ID 생성기 설계 | |
| 4 | Ch.08 | URL 단축기 설계 | |
| 4 | Ch.09 | 웹 크롤러 설계 | |
| 5 | Ch.10 | 알림 시스템 설계 | |
| 5 | Ch.11 | 뉴스 피드 시스템 설계 | |
| 6 | Ch.12 | 채팅 시스템 설계 | |
| 6 | Ch.13 | 검색어 자동완성 시스템 | |
| 7 | Ch.14 | 유튜브 설계 | |
| 7 | Ch.15 | 구글 드라이브 설계 | |
| 8 | Ch.16 | 배움은 끝이 없다 (마무리) | |

---

## 📁 폴더 구조

```
system-design-study/
├── README.md
├── ch01-규모-확장성/
│   ├── 발표자료.md
│   └── 토론정리.md
├── ch02-규모-추정/
├── ch03-면접-공략법/
├── ch04-처리율-제한/
├── ch05-안정-해시/
├── ch06-키값-저장소/
├── ch07-유일-ID-생성기/
├── ch08-URL-단축기/
├── ch09-웹-크롤러/
├── ch10-알림-시스템/
├── ch11-뉴스-피드/
├── ch12-채팅-시스템/
├── ch13-검색어-자동완성/
├── ch14-유튜브/
├── ch15-구글-드라이브/
└── ch16-마무리/
```

---

## 📝 스터리 자료 작성 가이드

### 발표자료에 포함할 것
- 챕터 핵심 내용 요약 (요구사항 → 설계 → 병목/개선 흐름 중심)
- 여러 설계안의 장단점 비교, 최종 선택 근거
- 본인이 인상 깊었던 부분 + 이유
- 실무 또는 프로젝트와 연결되는 지점(선택적)
- 토론하고 싶은 질문 1~2개(선택적)

### PR 규칙
- 브랜치: `ch{번호}-{이름}` (예: `ch04-incheol`)
- PR 제목: `[Ch.04] 처리율 제한 장치의 설계 — 정인철`

---

## 🤝 Ground Rules

1. **완독보다 이해가 우선** — 다 읽는 것보다 이해한 것을 나누는 게 중요합니다.
2. **"왜 이 설계인가"를 묻는다** — 정답을 외우기보다 트레이드오프를 따져봅니다.
3. **실무와 연결** — 책 내용을 본인 경험이나 프로젝트에 연결해봅니다.
4. **서로 존중** — 모르는 건 부끄러운 게 아닙니다. 함께 배우는 자리입니다.

---

## 📌 참고 자료
