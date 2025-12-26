**'기초(Hash/XOR) $\rightarrow$ 난제(ECC) $\rightarrow$ 상호작용 증명(Sigma/Schnorr) $\rightarrow$ 범용 증명(Circuit)'**

---

# [Week 1] 암호학의 레고 블록: XOR, Hash, and Commitment

**🎯 핵심 목표:**
1.  **XOR:** 왜 암호학에서 덧셈/곱셈보다 XOR을 사랑하는가? (Randomness vs Reversibility)
2.  **Commitment:** 해시와 머클트리가 데이터를 '확정'하고 '검증'하는 약속의 도구임을 이해한다.

### 1.1 The Magic of XOR (`^`)

**"무작위처럼 보이지만, 키만 있으면 언제든 돌아올 수 있다."**

*   **개발자 멘탈 모델:** `a ^ b = c` 일 때, `c`는 `a`와 `b`의 비트가 섞인 **난수**처럼 보입니다. 하지만 `c ^ b`를 하면 마법처럼 `a`가 복구됩니다.
*   **One-Time Pad (완전 보안의 기초):**
    *   데이터(Message)가 `0101`이고, 키(Key)가 랜덤한 `1100`이라면?
    *   암호문: `0101 ^ 1100 = 1001`
    *   해커가 `1001`을 봤을 때, 원본이 무엇일 확률은 모든 경우의 수에 대해 균일합니다. (완전한 Randomness)
*   **MPC/ZK에서의 활용:** 값을 숨기는(Blinding) 가장 기초적인 연산입니다.


### 1.2 Hash & Vector Commitment (Merkle Tree)

*   **Hash as Commitment:** "봉투에 넣고 밀봉하기". 내용을 바꾸지 않겠다고 약속(Binding)하고, 내용은 아직 안 보여줌(Hiding).
*   **Merkle Tree as Vector Commitment:**
    * 머클 트리는 **"벡터(배열)에 대한 커밋먼트"** 입니다.
    *   `Data = [tx1, tx2, ..., tx1000]`
    *   `Commitment = RootHash`
    *   **ZK 관점:** "내가 가진 `tx50`이 이 `RootHash` 안에 포함되어 있어!"를 증명할 때, 전체 배열을 다 깔 필요 없이 `log(N)` 개의 해시 경로(Witness)만 보여주면 됩니다.

---

# [Week 2] 타원곡선과 "되돌릴 수 없는 시간"

**🎯 핵심 목표:**
1.  **ECC:** 유한체 위에서 점을 더하는 행위가 왜 암호학적 난제가 되는가?
2.  **DLP:** $P = k \cdot G$는 쉬운데, $k = P / G$는 왜 불가능한가?

    <details><summary>Answer:</summary>

    > ▌ $P = k \cdot G$ (곱셈)은 순식간에 계산되지만, 그 역연산($\log_G{P}$)인 **"$P$와 $G$를 알 때 $k$ 구하기"** 는 우주의 수명이 다할 때까지 계산해도 불가능하다. (이산 로그 문제)

    </detalis>

---

# [Week 3] ZK의 시작: 시그마 프로토콜 (Sigma Protocols)

**🎯 핵심 목표:**
1.  **Schnorr 서명**이 사실은 가장 간단한 형태의 영지식 증명임을 깨닫는다.
2.  **Fiat-Shamir 변환**: 대화(Interaction)를 해시(Hash)로 대체하여 비대화형(Non-interactive)으로 만드는 마법.

### 3.1 시그마 프로토콜 ($\Sigma$-Protocols) 구조
시그마 프로토콜은 **3단계(Commit-Challenge-Response)**로 이루어진 증명입니다.

1.  **Commit ($A$):** 증명자(Prover)는 임의의 난수 $r$을 뽑아 커밋 $A = g^r$을 보냅니다. ("내가 주사위를 던져서 컵 안에 숨겼어.")
2.  **Challenge ($e$):** 검증자(Verifier)는 랜덤한 숫자 $e$를 퀴즈로 냅니다. ("그럼 컵을 흔들어 봐.")
3.  **Response ($z$):** 증명자는 비밀키 $x$와 난수 $r$, 퀴즈 $e$를 섞어 답 $z = r + e \cdot x$를 계산해 줍니다.
4.  **Verification:** 검증자는 $g^z$가 $A \cdot P^e$ 와 같은지 확인합니다. ($P$는 공개키)

### 3.2 이것이 왜 ZK인가? (Zero-Knowledge)
검증자가 보는 값은 $A, e, z$ 뿐입니다. 여기서 비밀키 $x$를 추출할 수 있을까요?
*   $z$는 $r$ (랜덤값)에 의해 가려져 있습니다(masked). $r$을 모르면 $z$에서 $x$를 분리할 수 없습니다. (XOR의 원리와 유사한 덧셈의 Blinding 효과)

---

# [Week 4] 범용 ZK: 코드를 다항식으로 (Circuits & Constraints)

**🎯 핵심 목표:**
1.  임의의 프로그램 실행을 증명하기 위해 **산술 회로(Arithmetic Circuit)** 로 변환하는 과정을 이해한다.
2.  **실용적인 다항식 예제**를 통해 R1CS와 QAP의 개념을 잡는다.

### 4.1 실용적인 예제: "피타고라스 정리 증명"

개발자들에게 "내가 직각삼각형의 변의 길이 $(a, b, c)$를 알고 있다"는 것을 $a, b$를 공개하지 않고 증명하는 상황을 가정합시다.

*   **코드:**
    ```python
    def check_right_triangle(a, b, c):
        return a*a + b*b == c*c
    ```
*   **산술 회로 (Gates):**
    1.  $v_1 = a \times a$
    2.  $v_2 = b \times b$
    3.  $v_3 = c \times c$
    4.  $out = v_1 + v_2 - v_3 = 0$

### 4.2 R1CS (Rank-1 Constraint System)
이것을 제약 조건 시스템(방정식 세트)으로 바꿉니다. $(A \cdot S) \times (B \cdot S) - (C \cdot S) = 0$ 형태입니다.

*   **Constraint 1:** $a \times a - v_1 = 0$
*   **Constraint 2:** $b \times b - v_2 = 0$
*   **Constraint 3:** $c \times c - v_3 = 0$
*   **Constraint 4:** $(v_1 + v_2) \times 1 - v_3 = 0$

### 4.3 다항식으로 변환 (Polynomial Interpolation)
이 제약 조건들을 하나의 거대한 다항식 $P(x)$로 만듭니다.
*   "만약 증명자가 올바른 $a, b, c$를 넣었다면, 이 다항식 $P(x)$는 특정 지점들(1, 2, 3, 4...)에서 0이 되어야 한다."
*   이제 문제는 **"이 다항식이 특정 점들에서 0이 됨을 증명하라"** 는 수학 문제로 바뀝니다.

