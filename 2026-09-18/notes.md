# 2026-09-18 · 웹 정찰 짧은 메모 (gobuster)

> 원본: [original/Notes_260918_144207_1.pdf](original/Notes_260918_144207_1.pdf)

분량이 매우 짧은 세션(한 줄 메모). 09-17 모의해킹 실습의 연장선.

## 메모
- 대상 `192.168.16.3` → **gobuster로 디렉터리 확인**.

```bash
sudo gobuster dir -u http://192.168.16.3 \
     -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

## 맥락
- 새 대상 IP를 잡으면 가장 먼저 하는 **웹 디렉터리 정찰** 단계.
- 자세한 nmap·gobuster·퍼징 사용법은 [2026-09-17 노트](../2026-09-17/notes.md) 참고.
