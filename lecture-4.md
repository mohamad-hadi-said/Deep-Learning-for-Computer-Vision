# 🔗 Backpropagation: الانتشار العكسي

## الفكرة الكبرى: "Circuit" حقيقي

تخيل إنك عم تمشي بـ **دارة كهربائية حقيقية** — كل "بوابة" (gate) بتحسب قيمة للأمام، وبترجع "إشارة" للخلف.

```

┌─────────────────────────────────────────────────────────────┐

│ CIRCUIT DIAGRAM │

│ │

│ x = -2 ──┐ │

│ ├──→ [+] ──→ q = 3 ──┐ │

│ y = 5 ───┘ (add) ├──→ [*] ──→ f = -12 │

│ ↑ (mul) │

│ z = -4 ───────────────────────┘ │

│ │

│ GREEN = Forward Pass (القيم) │

│ RED = Backward Pass (المشتقات) │

└─────────────────────────────────────────────────────────────┘

```

---

## 1️⃣ البوابات الأساسية (Gates)

### ➕ Add Gate (الجمع)

```

f(x,y) = x + y

∂f/∂x = 1

∂f/∂y = 1

```

**الترجمة البصرية:**

```

x ──┐

├──→ [+] ──→ z

y ──┘

dz/dx = 1 dz/dy = 1

```

> 🔑 **الـ Add Gate بـ "يوزع" الـ gradient بالتساوي على كل المدخلات**

---

### ✖️ Multiply Gate (الضرب)

```

f(x,y) = x * y

∂f/∂x = y

∂f/∂y = x

```

**الترجمة البصرية:**

```

x ──┐

├──→ [*] ──→ z

y ──┘

dz/dx = y dz/dy = x

```

> 🔑 **الـ Multiply Gate بـ "يقلب" المدخلات!** الـ gradient على x = y، وعلى y = x

---

### 🔝 Max Gate (الأقصى)

```

f(x,y) = max(x,y)

∂f/∂x = 1 if x >= y else 0

∂f/∂y = 1 if y >= x else 0

```

**الترجمة البصرية:**

```

x ──┐

├──→ [max] ──→ z

y ──┘

إذا x > y: dz/dx = 1, dz/dy = 0

إذا y > x: dz/dx = 0, dz/dy = 1

```

> 🔑 **الـ Max Gate بـ "يوجه" الـ gradient للمدخل الأكبر فقط**

---

## 2️⃣ مثال كامل: f(x,y,z) = (x+y)z

### Forward Pass (الأمام):

```

x = -2, y = 5, z = -4

q = x + y = -2 + 5 = 3

f = q * z = 3 * (-4) = -12

```

### Backward Pass (الخلف):

```

الهدف: نحسب ∂f/∂x, ∂f/∂y, ∂f/∂z

الخطوة 1: من f = q*z

∂f/∂q = z = -4

∂f/∂z = q = 3

الخطوة 2: من q = x+y

∂q/∂x = 1

∂q/∂y = 1

الخطوة 3: Chain Rule

∂f/∂x = ∂f/∂q * ∂q/∂x = (-4) * 1 = -4

∂f/∂y = ∂f/∂q * ∂q/∂y = (-4) * 1 = -4

∂f/∂z = 3

```

### الرسم البياني الكامل:

```

x = -2 ──┐ ∂f/∂x = -4

├──→ [+] ──→ q = 3 ──┐ ∂f/∂q = -4

y = 5 ───┘ ├──→ [*] ──→ f = -12

z = -4 ───────────────────────┘ ∂f/∂z = 3

```

> 💡 **الـ gradient على x سالب (-4)** → لو زدنا x شوي، f رح ينقص.

> **الـ gradient على z موجب (3)** → لو زدنا z شوي، f رح يزيد.

---

## 3️⃣ Sigmoid Neuron: مثال متقدم

```

f(w,x) = 1 / (1 + e^{-(w₀x₀ + w₁x₁ + w₂)})

```

### Forward Pass:

```

w = [2, -3, -3]

x = [-1, -2, 1] (x₂=1 هو bias)

dot = 2*(-1) + (-3)*(-2) + (-3)*1 = -2 + 6 - 3 = 1

f = 1 / (1 + e^{-1}) ≈ 0.73

```

### Backward Pass:

**السر الجميل:** مشتقة Sigmoid بتبسط لـ:

$$\frac{d\sigma}{dx} = \sigma(1-\sigma)$$

```

ddot = (1 - f) * f = (1 - 0.73) * 0.73 ≈ 0.20

dw₀ = x₀ * ddot = (-1) * 0.20 = -0.20

dw₁ = x₁ * ddot = (-2) * 0.20 = -0.40

dw₂ = x₂ * ddot = 1 * 0.20 = 0.20

dx₀ = w₀ * ddot = 2 * 0.20 = 0.40

dx₁ = w₁ * ddot = (-3) * 0.20 = -0.60

```

### الرسم البياني الكامل:

```

w₀=2 ───┐ dw₀ = -0.20

x₀=-1 ──┤ dx₀ = 0.40

├──→ [*] ──→ -2 ──┐

w₁=-3 ──┤ │

x₁=-2 ──┤ ├──→ [+] ──→ dot=1 ──→ [σ] ──→ f≈0.73

├──→ [*] ──→ 6 ──┘ ddot=0.20

w₂=-3 ──┤

x₂=1 ───┤

├──→ [*] ──→ -3 ──┘ dw₂ = 0.20

dx₂ = 0.20

```

> 🔑 **Pro Tip:** دائماً نستخدم **staged computation** — نكسر الحساب لخطوات وسيطة (مثل `dot`) ونحسب gradient لكل خطوة.

---

## 4️⃣ مثال معقد: Staged Computation

```

f(x,y) = (x + σ(y)) / (σ(x) + (x+y)²)

```

### Forward Pass (مرحلة بمرحلة):

```python

x = 3

y = -4

# (1) Sigmoid في البسط

sigy = 1.0 / (1 + math.exp(-y)) # σ(-4) ≈ 0.018

# (2) البسط

num = x + sigy # 3 + 0.018 ≈ 3.018

# (3) Sigmoid في المقام

sigx = 1.0 / (1 + math.exp(-x)) # σ(-3) ≈ 0.047

# (4) المقام

xpy = x + y # 3 + (-4) = -1

xpysqr = xpy**2 # (-1)² = 1

den = sigx + xpysqr # 0.047 + 1 = 1.047

# (5) القسمة

invden = 1.0 / den # 1/1.047 ≈ 0.955

f = num * invden # 3.018 * 0.955 ≈ 2.88

```

### Backward Pass (بالعكس تماماً!):

```python

# (8) f = num * invden

dnum = invden # ∂f/∂num = invden

dinvden = num # ∂f/∂invden = num

# (7) invden = 1.0 / den

dden = (-1.0 / (den**2)) * dinvden # ∂invden/∂den = -1/den²

# (6) den = sigx + xpysqr

dsigx = (1) * dden # ∂den/∂sigx = 1

dxpysqr = (1) * dden # ∂den/∂xpysqr = 1

# (5) xpysqr = xpy**2

dxpy = (2 * xpy) * dxpysqr # ∂xpysqr/∂xpy = 2*xpy

# (4) xpy = x + y

dx = (1) * dxpy # ∂xpy/∂x = 1

dy = (1) * dxpy # ∂xpy/∂y = 1

# (3) sigx = σ(x)

dx += ((1 - sigx) * sigx) * dsigx # 🔴 += مش = !!

# (2) num = x + sigy

dx += (1) * dnum # 🔴 += تاني!

dsigy = (1) * dnum

# (1) sigy = σ(y)

dy += ((1 - sigy) * sigy) * dsigy # 🔴 += تالت!

```

### 🔴 السر المهم: += مش =

**x و y ظهروا أكثر من مرة بالـ forward pass!**

```

x ──┐

├──→ [+] ──→ num

sigy┘

x ──┐

├──→ [+] ──→ xpy ──→ [²] ──→ xpysqr ──→ [+] ──→ den

y ──┘

x ──→ [σ] ──→ sigx ───────────────────────────────→ den

```

> 🔑 **قاعدة متعددة المتغيرات:** إذا متغير تفرع لأكثر من مكان، الـ gradients بتجمع!

$$\frac{\partial f}{\partial x} = \frac{\partial f}{\partial \text{num}} \cdot \frac{\partial \text{num}}{\partial x} + \frac{\partial f}{\partial \text{xpy}} \cdot \frac{\partial \text{xpy}}{\partial x} + \frac{\partial f}{\partial \text{sigx}} \cdot \frac{\partial \text{sigx}}{\partial x}$$

---

## 5️⃣ Patterns في Backward Flow

### الرسم البياني التوضيحي:

```

x=3 ───┐ dx = -8.00

├──→ [*] ──→ 6 ───┐

y=-4 ──┘ │

├──→ [max] ──→ 2 ───┐

z=2 ───┐ │ │

├──→ [max] ──→ 2 ─┘ ├──→ [+] ──→ -10 ──→ [*] ──→ -20

w=-1 ──┘ ↑

│

2 ────────────────────────────────────────────┘

d(max)/dz = 2 (z > w, فالـ gradient بيروح لـ z)

d(max)/dw = 0 (w < z, فالـ gradient = 0)

d(*)/dx = -4 * 2 = -8 (قلب المدخلات!)

d(*)/dy = 3 * 2 = 6

```

| البوابة | Forward | Backward | الترجمة البصرية |

|---------|---------|----------|----------------|

| **Add** | z = x + y | dz/dx = 1, dz/dy = 1 | "يوزع" الـ gradient بالتساوي |

| **Max** | z = max(x,y) | dz/dx = 1(x≥y), dz/dy = 1(y≥x) | "يوجه" الـ gradient للأكبر |

| **Mul** | z = x * y | dz/dx = y, dz/dy = x | "يقلب" المدخلات |

---

## 6️⃣ Vectorized Operations

### Matrix-Matrix Multiply:

```python

# Forward

W = np.random.randn(5, 10) # (5, 10)

X = np.random.randn(10, 3) # (10, 3)

D = W.dot(X) # (5, 3)

# Backward (نفترض عندنا dD)

dD = np.random.randn(5, 3) # نفس shape كـ D

# 🔑 الطريقة: تحليل الأبعاد!

dW = dD.dot(X.T) # (5,3) · (3,10) = (5,10) ✓ نفس W!

dX = W.T.dot(dD) # (10,5) · (5,3) = (10,3) ✓ نفس X!

```

> 💡 **Tip:** ما تحفظ الصيغ! استخدم **dimension analysis** — شوف شو الأبعاد اللي بدك إياها، والباقي بيجي لوحده.

---

## 7️⃣ الخلاصة: Backpropagation كـ "Circuit"

```

┌─────────────────────────────────────────────────────────────┐

│ FORWARD PASS │

│ │

│ x ──→ [gate1] ──→ [gate2] ──→ [gate3] ──→ ... ──→ f │

│ ↑ local grad ↑ local grad │

│ │

│ كل gate بتحسب: │

│ 1. output value (الأمام) │

│ 2. local gradient (الخلف) │

│ │

├─────────────────────────────────────────────────────────────┤

│ BACKWARD PASS │

│ │

│ df/dx ← [gate1] ← [gate2] ← [gate3] ← ... ← df/df = 1 │

│ ↑ ↑ │

│ chain rule chain rule │

│ │

│ كل gate بتاخد: │

│ 1. gradient من output (من اليمين) │

│ 2. local gradient (من forward pass) │

│ 3. بتضربهم (chain rule) وبتمرر لليسار │

│ │

└─────────────────────────────────────────────────────────────┘

```

---

## 🎯 النقاط اللي مستحيل تاخدها من الفيديو

| النقطة | الشرح |

|--------|-------|

| **Backprop = Local Process** | كل gate بتحتاج بس مدخلاتها ومخرجاتها — ما بتهم بالـ circuit الكامل |

| **Add Gate = Distributor** | بيوزع gradient بالتساوي |

| **Max Gate = Router** | بيوجه gradient للمدخل الأكبر فقط |

| **Mul Gate = Swapper** | بيقلب المدخلات |

| **+= للمتغيرات المتفرعة** | إذا x ظهر أكثر من مرة، gradients بتجمع |

| **Staged Computation** | دائماً نكسر لخطوات وسيطة |

| **Dimension Analysis** | ما نحفظ — نحلل الأبعاد |

| **Caching** | نحفظ قيم forward لاستخدامها بال backward |

> **Backpropagation = Chain Rule + Local Gradients + Backward Flow**