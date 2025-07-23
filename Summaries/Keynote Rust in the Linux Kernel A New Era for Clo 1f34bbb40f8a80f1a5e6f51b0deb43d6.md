# Keynote: Rust in the Linux Kernel: A New Era for Cloud Native Performance and... Greg Kroah-Hartman

다중 선택: KubeCon 2025 EU

# Rust와 Linux

## 1. Linux 커널 현황

- **역할**: 하드웨어 추상화·프로세스 격리·안정성 제공
- **규모**
    - 약 700명 유지보수자, 3,000명 이상 기여자
    - 연간 76,000개 커밋 → 시간당 8~9개 승인 (24×7, 지난 10년간)
    - 8~9주 주기 릴리스, 1,300여 개 Stable 릴리스
- **보안 이슈**
    - 하루 평균 13건 CVE 발생(타 OS 대비 절반 수준)
    - 유지보수자 리뷰가 평균 3회 필요

## 2. C의 한계와 보완

- **잠금 해제/메모리 해제 버그**
    - `goto` 기반 에러 처리 → 언락(Unlock) 누락
    - 수동 `free()` → 해제 누락
- **C 언어 확장**
    - **Scoped references** (C23) 도입
        
        ```c
        {
          scoped_ref mlxguard_t ml = take_mutex(&obj->lock);
          … // lock 자동 해제
        }
        
        ```
        
    - **Scoped alloc**
        
        ```c
        {
          scoped_ptr int *p = kmalloc(...);
          … // scope 벗어나면 자동 kfree()
        }
        
        ```
        

## 3. Rust의 장점

- **컴파일 시점 검증**
    - 에러 반환값(‵?‵) 자동 처리 → 누락 불가
    - 메모리 수명·잠금 규칙 강제 → 사용 오류 사전 차단
    - 타입 안전성↑ → 잘못된 메모리 접근 시 안전하게 패닉
- **코드 간결·가독성↑**
    - `unsafe` 범위 최소화, 일반 로직은 안전 Rust로 작성
    - 리뷰 시간 절감 → 유지보수 부담↓
- **실제 적용 현황**
    - 2백만 LOC(C)에 대조, 신규 GPU 드라이버 등 25천 LOC Rust
    - 커널 내 Rust 수용 → 점진적 확장 중

## 4. 도전 과제

- **기존 C 코드와의 융합**
    - API 호환성 유지
    - 오래된 코드 재검토 필요
- **개발자·유지보수자 학습 곡선**
    - Rust 문화·도구체인 숙련 요구

## 5. 결론

- Rust 도입은 “완벽한” 해법이 아니지만
    - “더 안전하게” 실패(panic)하도록 돕고
    - 컴파일러가 보안·안정성 검증을 수행 → 리뷰 부담↓
- 장기적으로 Linux 커널 안정성·유지보수성↑
- “C와 Rust, 둘 다 쓰되 적재적소에”가 현명한 전략

---

**참고**

- Linux 6.x 커널의 Rust 지원: `–enable=rust` 옵션
- C23 scoped refs/allocs: [WG14 논의](https://en.cppreference.com/w/c/memory/scoped_ref)
- Rust-for-Linux 프로젝트: [https://github.com/Rust-for-Linux/linux-rust](https://github.com/Rust-for-Linux/linux-rust)