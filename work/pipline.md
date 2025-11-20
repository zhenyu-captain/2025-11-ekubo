## 目标
- Wolfram Mathematica 目标：数学公式的快速验证和问题发现。通过高精度数学计算和符号计算，在 Ekubo 项目中大规模测试各种边界条件，识别可疑区间和潜在问题，并通过可视化和数值分析定位错误
- Coq 目标：对已发现问题进行严格的形式化证明。通过定理证明，验证 Ekubo 项目中关键数学函数的特定性质（如单调性、可逆性、误差边界），并证明代码实现与数学模型在已定义的规范下的一致性。
- PoC 目标：验证漏洞的可行性与利用性，PoC 旨在通过 展示漏洞 和 利用示例，验证漏洞是否 真实存在，并帮助 团队 和 审计员 理解漏洞的 实际可操作性。


## Prompt1: 扫描 Ekubo 仓库，生成文件列表与函数列表（Hotspot Map）
目标：扫描整个 Ekubo 仓库，按以下类别生成文件列表与函数列表（Hotspot Map），提取所有涉及数学计算的函数和汇编块，并提供每个函数的简要描述。
```
请扫描整个 Ekubo 仓库，按照以下类别生成文件列表与函数列表（Hotspot Map）：
1. Tick 与 Price 相关数学函数（如 tickToPrice、priceToTick、sqrtPriceX96 等）
2. Liquidity math（liquidity delta、amount0/amount1、跨 tick 逻辑）
3. TWAMM math（time-weighted 平均价格、订单执行、连续/离散转换、虚拟储备）
4. Fee growth math（feeGrowthGlobal0/1、fee 序列、fee update）
5. 所有 inline assembly block（记录文件名、函数名、assembly 行号）

每个分类包含：
- file（文件路径）
- function name（函数名）
- short description of math involved（数学涉及的简短描述）

**输出结果**：请生成一个 JSON 文件，如map.json，列出每个分类中的所有文件、函数及其简要描述。
```


## Prompt2：从代码提取数学公式并验证
目标：从 Prompt1 生成的 map.json 文件中标记的 sqrtRatioToTick 回到 Solidity 代码提取数学公式。


## 人为介入流程
### 确定 wolfra 公式
``` mathematica
(* ============================================= *)
(* 完整实现 sqrtRatioToTick 数学模型的验证 *)
(* ============================================= *)

(* 1. 定义关键常量和辅助函数 *)

(* log2(1.0001) 的常量 *)
log2OnePoint0001 = Log2[1.0001];

(* 误差边界 (Q128.128 格式) *)
errorBoundX128 = 702958256323748294;

(* 对数计算的常数 *)
K_2_OVER_LN2_X64 = 53226052391377289966;

(* invLb，log2(sqrt(1.0001)) 的倒数 *)
INV_LB_X64 = 25572630076711825471857579;

(* 2. 定义 sqrtRatioToTick 函数 *)

sqrtRatioToTick[sqrtRatio_] := Module[{R_f, logBaseTick128, logBase128, tickLower, tickUpper, tickFinal},
  
  (* 转换 sqrtRatio 为定点数 (Q64.128 格式) *)
  R_f = sqrtRatio * 2^128;

  (* 计算 log2(sqrtRatio) *)
  logBaseTick128 = Log2[R_f^2] / log2OnePoint0001; 

  (* 对 logBaseTick128 进行舍入操作，应用误差边界 *)
  tickLower = Floor[(logBaseTick128 - errorBoundX128) / 2^128];
  tickUpper = Floor[(logBaseTick128 + errorBoundX128) / 2^128];
  
  (* 根据舍入边界决定最终的 tick 值 *)
  tickFinal = If[Abs[tickUpper - tickLower] >= 2, tickLower, tickUpper];
  
  (* 返回计算结果 *)
  tickFinal
]

(* 3. 验证过程：测试不同情况 *)

(* 示例：计算 sqrtRatio = 2^64 对应的 tick 值 *)
sqrtRatio = 2^64;
tickResult = sqrtRatioToTick[sqrtRatio];
Print["tick for sqrtRatio = 2^64: ", tickResult]

(* 验证单调性：sqrtRatio 增加时，tick 是否递增 *)
sqrtRatioIncrease = 2^64 + 1;
tickResultIncrease = sqrtRatioToTick[sqrtRatioIncrease];
Print["tick for sqrtRatio = 2^64 + 1: ", tickResultIncrease]

(* 验证连续性：sqrtRatio 接近 2^128，tick 是否连续 *)
sqrtRatioBoundary = 2^128;
tickResultBoundary = sqrtRatioToTick[sqrtRatioBoundary];
Print["tick for sqrtRatio = 2^128: ", tickResultBoundary]

(* 测试溢出保护：当 sqrtRatio 非常大时 *)
sqrtRatioLarge = 2^96; (* 设置一个非常大的值来测试溢出保护 *)
tickResultLarge = sqrtRatioToTick[sqrtRatioLarge];
Print["tick for large sqrtRatio: ", tickResultLarge]

(* 4. 验证计算结果 *)
(* 理论上，sqrtRatioToTick 计算的 tick 值应该与理想 tick 值相符 *)
expectedTick = Floor[Log[sqrtRatio^2, 1.0001]];  (* 理想 tick 计算公式 *)
Print["Expected tick: ", expectedTick]
```

### 确定 wolfra 公式转译的 coq 公式
``` coq

```

