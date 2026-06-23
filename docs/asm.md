# 《汇编语言程序设计与调试》考前练习

??? note "参考资料"
    > **参考资料**
    >
    > - **2026-06-17 第11-13节 课堂录音识别稿**
    > - `汇编复习1.asm` 补充知识点
    > - 课程讲义（第2-16周）示例程序

---

<span style="font-size:1.2em; color:#808080;">**更新日志**</span>

=== "2026-06-23"
 按考试形式（判断+单选）整理，覆盖课堂重点：标志位、`jl`(SF≠OF)、字符串指令、三种调用（call/call far/int）的 push 顺序、中断编程（int 8h/中断向量表/hook 流程）、堆栈框架、调试技巧等。答案标注 `[推测]`，未经人工验证。

---

## 判断题

!!! question "1."
    `mov ax, 0` 执行后，零标志 ZF 会变为 1。（）

    ??? info "点击查看答案"
        **✗（错误）**。课堂反复强调：`mov` **不影响任何标志位**。AX 变成 0 但 ZF 不变。要让 ZF=1 必须用 `sub ax, ax` 或 `xor ax, ax`。

!!! question "2."
    `jl`（有符号小于则跳）的跳转条件是 SF ≠ OF，与 ZF 的值无关。（）

    ??? info "点击查看答案"
        **✓（正确）**。调试器演示：比较 1 和 2（SF=1,OF=0），手动把 ZF 改成 1，`jl` 箭头依然亮——**只看 SF 和 OF 是否不等**，无视 ZF。

!!! question "3."
    两个正数相加变成负数时 OF=1，两个负数相加变成正数时 OF=1。（）

    ??? info "点击查看答案"
        **✓（正确）**。举例：7Fh(127)+1=80h(-128)→OF=1；80h(-128)+0FFh(-1)=7Fh(+127)→OF=1。OF 是 CPU 假设操作数为**有符号数**设置的，非符号数的溢出看 CF。

!!! question "4."
    `repne scasb` 和 `loop` 对 CX 的处理方式相同：CX=0 时都循环 10000h 次。（）

    ??? info "点击查看答案"
        **✗（错误）**。需注意：`repne` **先判断** CX=0，为 0 则 0 次循环；`loop` **先 dec 再判断**，CX=0 时循环 10000h 次。二者相反。

!!! question "5."
    `cmp` 与 `sub` 的区别在于 `cmp` 不保存运算结果但影响标志位。（）

    ??? info "点击查看答案"
        **✓（正确）**。`cmp ax, bx` 等价 `sub ax, bx` 但不写回 AX。`cmp` 后跟 `je` 等于 `sub` 后跟 `jz`。

!!! question "6."
    短跳（short jump）用 delta 编码而非绝对地址，目的是代码被复制到其他位置后仍能正确跳转。（）

    ??? info "点击查看答案"
        **✓（正确）**。`EB 06` 中 06 = 目标 − 下条指令 = delta。代码从 1000:2000 复制到 1000:8000 后仍跳到正确相对位置。近跳（E9）用 2 字节 delta。

!!! question "7."
    `test ah, 1` 和 `and ah, 1` 的唯一区别是前者不保存结果，其余行为完全相同。（）

    ??? info "点击查看答案"
        **✓（正确）**。`test` 做按位与但丢弃结果，仅影响标志位；`and` 写回目的操作数。

!!! question "8."
    `neg ax` 等价于 `not ax` 后 `inc ax`。（）

    ??? info "点击查看答案"
        **✓（正确）**。`neg ax = (not ax) + 1`，结果值相同。唯一差异在 CF：`neg` 对非零操作数置 CF=1（内部做 0−AX 产生借位），`inc` 不影响 CF。

        **举例**：设 AX=2，初始 CF=0。
        - `neg ax`：AX=0FFFEh(−2)，**CF=1**（2≠0，0−2 有借位）
        - `not ax; inc ax`：AX=0FFFEh(−2)，**CF=0**（inc 不改 CF）

        二者 AX 结果相同，CF 不同。

!!! question "9."
    `sub` 指令执行后，可以直接跟 `js` 判断结果是否为负，不需要中间再写 `cmp`。（）

    ??? info "点击查看答案"
        **✓（正确）**。老师强调：`sub` 之后已影响标志位，马上可以判断。再写 `cmp` 是“啰嗦”“没学透”。

!!! question "10."
    `cbw` 的典型用途是为 `idiv` 准备被除数，把 AL 符号扩展到 AX。（）

    ??? info "点击查看答案"
        **✓（正确）**。`cbw`/`cwd`/`cdq` 的“意图”就是为 `idiv` 服务。`cwd` 把 AX 扩到 DX:AX，正是为 16 位 `idiv` 做准备。

!!! question "11."
    除法溢出（如除数为 0）产生 `int 0` 中断，该中断插在 `div` 指令的**下方**。（）

    ??? info "点击查看答案"
        **✗（错误）**。注意：`int 0` 插在 `div` 指令的**上方**（前面）。“必然是 div 的上方会插到 int 0，不是在下面的”。

!!! question "12."
    在 `mov ax, [bp+2]` 中，缺省的段寄存器是 DS。（）

    ??? info "点击查看答案"
        **✗（错误）**。`[]` 中有 BP 时，缺省段寄存器是 **SS**（堆栈段）；无 BP 只有 BX/SI/DI 时才是 DS。

---

## 填空题

!!! question "1."
    十进制数 -12 的 8 位二进制补码为 ______ B。

    ??? info "点击查看答案"
        **1111 0100**。12=00001100B，取反加一。

!!! question "2."
    逻辑地址 1234h:0058h 对应的物理地址为 ______ h。

    ??? info "点击查看答案"
        **12398h**。物理地址 = 段地址×10h + 偏移 = 12340h + 0058h。

!!! question "3."
    设 AL=00h，执行 `sub AL, 01h` 后，CF= ______。

    ??? info "点击查看答案"
        **1**。0−1 产生借位。

!!! question "4."
    从地址 1000h:2000h 开始顺序存放 12h, 34h, 56h, 78h 四个字节，则 `word ptr [1000h:2002h]` = ______ h。

    ??? info "点击查看答案"
        **7856h**。小端序：2000→12h, 2001→34h, 2002→56h(低), 2003→78h(高)。

!!! question "5."
    近调用 `call` 压栈 ______ 个值（仅压入 ______）；远调用 `call far` 压栈 ______ 个值（先 ______ 后 ______）；`int` 压栈 ______ 个值（顺序为 ______ → ______ → ______）。

    ??? info "点击查看答案"
        **1**、**IP**；**2**、**CS**、**IP**；**3**、**FLAGS**、**CS**、**IP**。返回指令对应 `ret`、`retf`、`iret`。总结：“一个 push、两个 push、三个 push 的区别”。

!!! question "6."
    `neg ax` 与 `not ax` 的关系：neg ax = (not ax) ______。

    ??? info "点击查看答案"
        **+ 1**。`-ax = ~ax + 1` 恒成立。

!!! question "7."
    在 `push bp; mov bp, sp` 建立的堆栈框架中（near call + Pascal 方式），`[bp+0]`=______，`[bp+2]`=______，`[bp+4]`=______。

    ??? info "点击查看答案"
        **old BP**、**返回地址(IP)**、**最后一个参数**。参数用 BP+N 引用，局部变量用 BP−N 引用。

!!! question "8."
    `jl` 的跳转条件为 ______（用标志位表达式），该指令 ______（会/不会）受 ZF 影响。

    ??? info "点击查看答案"
        **SF ≠ OF**、**不会**。调试器验证：即使 ZF=1，`jl` 仍只看 SF 和 OF。

---

## 单选题

!!! question "1."
    设 AL=7Fh，执行 `add AL, 1` 后，SF 和 OF 的值分别是（）

    (A) SF=0, OF=0

    (B) SF=0, OF=1

    (C) SF=1, OF=0

    (D) SF=1, OF=1

    ??? info "点击查看答案"
        **D（SF=1, OF=1）**。7Fh+1=80h，最高位=1→SF=1；127+1=−128，正+正→负→OF=1。

!!! question "2."
    设 AL=0FFh，执行 `add AL, 1` 后，CF 和 OF 的值分别是（）

    (A) CF=0, OF=0

    (B) CF=1, OF=1

    (C) CF=1, OF=0

    (D) CF=0, OF=1

    ??? info "点击查看答案"
        **C（CF=1, OF=0）**。0FFh+1=100h，8 位溢出→CF=1。但 −1+1=0，正负相加永不溢出→OF=0。此例说明 CF 是非符号数溢出，OF 是有符号数溢出。

!!! question "3."
    `repne scasb` 扫描 "Hello\0"（6字节）结束后，要得到字符串长度 6，应对 CX 做（）

    (A) `inc cx`

    (B) `dec cx`

    (C) `not cx`

    (D) `neg cx`

    ??? info "点击查看答案"
        **C（not cx）**。初始 CX=0FFFFh，扫描 6 次后 CX=0FFF9h，`not cx` = 0006h = 6。这是巧妙替代减法。

!!! question "4."
    关于 `sal` 和 `shl` 的说法正确的是（）

    (A) `sal` 是算术左移，`shl` 是逻辑左移，行为不同

    (B) `sal` 和 `shl` 完全等价

    (C) `sal` 保持符号位不变

    (D) `sal` 不能用于无符号数

    ??? info "点击查看答案"
        **B（完全等价）**。讲义中明确：`sal` ≡ `shl`，左移时算术和逻辑行为一致。

!!! question "5."
    设 AL=0FEh（−2），执行 `sar AL, 1` 和 `shr AL, 1` 的结果分别是（）

    (A) 0FFh, 7Fh

    (B) 7Fh, 0FFh

    (C) 0FFh, 0FFh

    (D) 7Fh, 7Fh

    ??? info "点击查看答案"
        **A（0FFh, 7Fh）**。`sar`：高位用符号位(1)填充→11111111=0FFh=−1；`shr`：高位用 0 填充→01111111=7Fh=127。对应 C 语言 `signed char` 和 `unsigned char` 右移差异。

!!! question "6."
    关于 `lodsb` 和 `stosb`，以下说法错误的是（）

    (A) `lodsb` = AL←DS:[SI], SI±1（DF=0 时递增）

    (B) `stosb` = ES:[DI]←AL, DI±1（DF=0 时递增）

    (C) DF=1 时 SI/DI 递减

    (D) `lodsb` 使用目标指针 ES:DI

    ??? info "点击查看答案"
        **D**。注意：`lodsb` 从**源**取数用 DS:SI；`stosb` 存到**目标**用 ES:DI。源和目标不能混。

!!! question "7."
    `xlat` 指令的功能是（）

    (A) AL = DS:[BX + SI]

    (B) AL = DS:[BX + AL]

    (C) AL = ES:[DI + AL]

    (D) AL = DS:[SI]

    ??? info "点击查看答案"
        **B（AL = DS:[BX + AL]）**。以 AL 为下标、DS:BX 为基址查表，结果放回 AL。

!!! question "8."
    近调用 `call`（near call）执行时，CPU 将什么压入堆栈？（）

    (A) 只压入 FLAGS

    (B) 只压入 IP（返回偏移地址）

    (C) 先压入 CS 再压入 IP

    (D) 先压入 FLAGS 再 CS 再 IP

    ??? info "点击查看答案"
        **B（只压入 IP）**。总结：近调用 = 1 个 push(IP) + jmp。对应 `ret`（pop IP）。

!!! question "9."
    远调用 `call far` 执行时，CPU 的压栈顺序是（）

    (A) 先压入 IP，再压入 CS

    (B) 先压入 CS，再压入 IP

    (C) 先压入 FLAGS，再压入 CS，最后 IP

    (D) 只压入 CS

    ??? info "点击查看答案"
        **B（先 CS，再 IP）**。远调用 = 2 个 push(CS,IP) + jmp。压栈顺序 CS 先、IP 后，返回时 `retf` 先 pop IP 再 pop CS。

!!! question "10."
    `int` 指令执行时，CPU 的压栈顺序是（）

    (A) FLAGS → CS → IP

    (B) IP → CS → FLAGS

    (C) CS → IP → FLAGS

    (D) FLAGS → IP → CS

    ??? info "点击查看答案"
        **A（FLAGS → CS → IP）**。总结：`int` = 3 个 push + jmp。顺序是 FLAGS 先、CS 次之、IP 最后。对应 `iret`（pop IP→CS→FLAGS）。

!!! question "11."
    关于 `call` / `call far` / `int` 三种调用，以下正确的是（）

    (A) 三者压栈内容完全一样

    (B) `call` 压 1 个值，`call far` 压 2 个值，`int` 压 3 个值

    (C) `int` 不压入 FLAGS

    (D) 三者都用 `ret` 返回

    ??? info "点击查看答案"
        **B**。总结：`call` = 1 push (IP)；`call far` = 2 push (CS,IP)；`int` = 3 push (FLAGS,CS,IP)。返回指令对应 `ret`、`retf`、`iret`。

!!! question "12."
    在堆栈框架 `push bp; mov bp, sp` 之后（near call + Pascal 方式），`[bp+4]` 指向（）

    (A) old BP

    (B) 返回地址（back / IP）

    (C) 最后一个参数

    (D) 第一个局部变量

    ??? info "点击查看答案"
        **C（最后一个参数）**。说“倒背如流”：`[bp+0]`=old BP, `[bp+2]`=返回地址, `[bp+4]`=最后一个参数。用 BP+N 引用参数，BP−N 引用局部变量。

!!! question "13."
    关于 C 语言和 Pascal 语言调用约定的区别，正确的是（）

    (A) C：参数从左到右压栈，调用者清理

    (B) C：参数从右到左压栈，调用者清理

    (C) Pascal：参数从右到左压栈，被调用者用 `ret n` 清理

    (D) Pascal：参数从右到左压栈，调用者清理

    ??? info "点击查看答案"
        **B**。C：从右到左 + 调用者清理（`add sp, n`）；Pascal：从左到右 + 被调用者清理（`ret n`）。C 从右到左是为支持 `printf` 可变参数。

!!! question "14."
    DOS 中断 `int 21h` 中，AH=9 的功能是（）

    (A) 显示字符，入口 DL=字符

    (B) 键盘输入，出口 AL=字符

    (C) 显示以 `$` 结尾的字符串，入口 DS:DX→字符串

    (D) 退出程序，入口 AL=返回码

    ??? info "点击查看答案"
        **C**。AH=1 键盘输入、AH=2 显示字符（DL）、AH=9 显示 `$` 结尾串（DS:DX）、AH=4Ch 退出。

!!! question "15."
    8086 的中断向量表位于内存何处？（）

    (A) 0:0 ~ 0:3FFh

    (B) 40:0 ~ 40:3FFh

    (C) F000:0 ~ F000:FFFFh

    (D) B800:0 ~ B800:FFFFh

    ??? info "点击查看答案"
        **A（0:0 ~ 0:3FFh）**。256 个中断向量，每个 4 字节（IP:CS），占 400h 字节。int 8h 向量在 8×4=20h 处。

!!! question "16."
    修改中断向量（hook int 8h）的正确流程是（）

    (A) 直接写入新地址即可

    (B) `cli` → 写入新向量 → `sti`

    (C) 保存旧向量 → `cli` → 写入新向量 → `sti` → 退出前恢复旧向量

    (D) 保存旧向量 → 写入新向量 → `iret` 返回

    ??? info "点击查看答案"
        **C**。int 8h 示例的完整流程：(1)保存旧向量；(2)`cli`；(3)写新向量到 0:8×4；(4)`sti`；(5)退出前恢复旧向量。`cli`/`sti` 之间操作须原子化防止竞态。

!!! question "17."
    调试中跟踪到 `call` 指令，想进入被调用函数内部应用（）

    (A) F7（trace into）

    (B) F8（step over）

    (C) F9

    (D) F5

    ??? info "点击查看答案"
        **A（F7）**。F7=trace into（进入函数），F8=step over（跳过）。调试 `repne scasb` 等也必须用 F7 才能逐步观察，F8 一步执行完。

!!! question "18."
    关于硬件断点，正确的是（）

    (A) `bpm x` 在变量被读取时中断

    (B) `bpm r` 在变量被执行时中断

    (C) `bpm w` 在变量被写入时中断

    (D) 8086 不支持硬件断点

    ??? info "点击查看答案"
        **C（bpm w 在写入时中断）**。注意：`bpm x`=执行断点、`bpm r`=读断点、`bpm w`=写断点。三者必须分清。

---


---

## 附录：核心速查

??? info "三种调用对比（call / call far / int）"

    | | near call | far call | int |
    |:--|:--:|:--:|:--:|
    | push 数量 | 1 | 2 | 3 |
    | push 顺序 | IP | CS → IP | FLAGS → CS → IP |
    | 返回指令 | `ret` | `retf` | `iret` |
    | 返回时 pop | IP | IP → CS | IP → CS → FLAGS |

    > 课堂原话：近调用 = 一个 push + 一个 jmp；远调用 = 两个 push + 一个 jmp；int = 三个 push + 一个 jmp。顺序不能错。

??? info "堆栈框架（near call + Pascal 方式）"

    ```
    SS:0FF8 old bp ← BP 指向此处 ([bp+0])
    SS:0FFA back ← 返回地址 IP ([bp+2])
    SS:0FFC 参数2 ← [bp+4]
    SS:0FFE 参数1 ← [bp+6]
    SS:1000 ← 初始 SP
    ```

    > 口诀：`[bp+0]`=old_bp，`[bp+2]`=返回地址，`[bp+N]`(N≥4)=参数。BP+N 引用参数，BP−N 引用局部变量。

??? info "中断向量与 Hook 流程"

    1. 中断向量表位于 **0:0 ~ 0:3FFh**（256个×4字节）
    2. int N 的向量地址 = 0:N×4
    3. Hook int 8h 流程：保存旧向量 → `cli` → 写新向量到 0:8×4 → `sti` → 退出前恢复旧向量

??? info "DOS 中断"

    1. int 21h

    | AH | 功能 | 入口 | 出口 |
    |:--:|------|------|------|
    | 1 | 键盘输入+回显 | — | AL=字符 |
    | 2 | 显示字符 | DL=字符 | — |
    | 9 | 显示 `$` 结尾串 | DS:DX→串 | — |
    | 4Ch | 退出 | AL=返回码 | — |

    2. int 00h：**除法溢出**（除数为 0 或商超出寄存器范围时触发，插在 `div`/`idiv` 指令上方）

    3. int 01h：**单步调试**（TF=1 时每条指令后自动触发，用于调试器单步执行）

    4. int 08h：**定时器中断**（硬件时钟，约每秒 18.2 次，常用于延时和周期性任务）

    5. int 09h：**键盘中断**（按键时触发，可 hook 来拦截键盘输入）

    > 课堂涉及：int 00h（div 溢出）、int 08h（hook 定时器实现 delay_1s、pclife 游戏外挂）、int 09h（“英特9”键盘中断）。

??? info "标志位速查（AF、PF 不作考试要求）"

    | 标志 | 说明 | 相关指令 |
    |:--:|------|------|
    | CF | 进位/借位/移位末位 | `clc`,`stc`,`jc`≡`jb` |
    | ZF | 结果为零 | `jz`≡`je` |
    | SF | 结果最高位 | `js`; `jl` 条件之一 |
    | OF | 有符号溢出 | `jo`; **`jl`=SF≠OF** |
    | IF | 中断允许 | `cli`(关), `sti`(开) |
    | DF | 串方向 | `cld`(DF=0,正向), `std`(DF=1,逆向) |

??? info "条件跳转核心"

    | 指令 | 条件 | 备注 |
    |:--|:--|:--|
    | **`jl`** | **SF ≠ OF** | **无视 ZF！调试器验证** |
    | `jg` | SF=OF 且 ZF=0 | |
    | `jb`/`jc` | CF=1 | 等价 |
    | `je`/`jz` | ZF=1 | 等价 |
    | `loop` | CX←CX−1, ≠0 跳 | CX=0 时循环 10000h 次 |
    | `repne` | CX=0 先判,0 次循环 | **与 loop 相反！** |

??? info "字符串指令速查"

    | 指令 | 功能 | 寄存器 |
    |:--|:--|:--|
    | `repne scasb` | 扫描找 AL，等效 strlen | ES:DI, CX, AL |
    | `rep movsb` | 复制 CX 字节 | DS:SI→ES:DI, CX |
    | `lodsb` | AL=DS:[SI], SI±1 | DS:SI |
    | `stosb` | ES:[DI]=AL, DI±1 | ES:DI |
    | `xlat` | AL=DS:[BX+AL] | DS:BX |

    > 方向由 DF 控制：`cld`→递增，`std`→递减。

??? info "调试器速查"

    | 操作 | 按键/命令 |
    |:--|:--|
    | trace into（进入函数） | F7 |
    | step over（跳过函数） | F8 |
    | 硬件执行断点 | `bpm x` |
    | 硬件读断点 | `bpm r` |
    | 硬件写断点 | `bpm w` |

??? info "间接寻址缺省段址"

    | 寻址方式 | 缺省段寄存器 |
    |:--|:--:|
    | `[bx]`, `[si]`, `[di]` | DS |
    | **`[bp]`** | **SS** |
    | `[bx+si]`, `[bx+di]` | DS |
    | `[bp+si]`, `[bp+di]` | SS |

