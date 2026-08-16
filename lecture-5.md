# 🖼️ المحاضرة 5: تصنيف الصور باستخدام الشبكات التلافيفية (CNNs)

## 1️⃣ مراجعة سريعة: المشكلة مع Linear Classifiers

### Linear Classifier = ضعيف جداً

| المشكلة | التوضيح |

|---------|---------|

| **Visual Viewpoint** | كل فئة = template واحد ضبابي (شايف الصور بالـ slide 9؟) |

| **Geometric Viewpoint** | يقدر يرسم بس حدود خطية — ما بيقدر يفصل بيانات معقدة |

```

f(x, W) = Wx + b

```

> الصور بـ 32×32×3 = 3072 بكسل. لما نبسطها لمتجه واحد (flatten)، **بنضيع البنية المكانية (Spatial Structure)** للصورة! (slide 28)

---

## 2️⃣ الحل: Convolutional Neural Networks (CNNs)

### الفكرة الأساسية: **الحفاظ على البنية المكانية**

بدل ما نبسط الصورة لمتجه، نتركها **3D** ونطبق عليها "فلاتر" (Filters) صغيرة:

```

Input: 32×32×3 (height × width × depth/channels)

↓

[5×5×3 Filter] ← ينزلق (slides) على الصورة

↓

Output: 28×28×1 (Activation Map)

```

### كيف يعمل الـ Convolution؟ (slides 49-56)

**الفلتر = مصفوفة صغيرة من الأوزان** (مثلاً 5×5×3)

```

لكل موقع مكاني (x,y) بالصورة:

1. ناخذ قطعة 5×5×3 من الصورة

2. نضربها element-wise بالفلتر

3. نجمع النتيجة + bias

4. هاي القيمة = عنصر واحد بالـ Activation Map

```

> **الفلتر ينزلق (slides) على كل الصورة** — نفس الأوزان لكل المواقع! (Parameter Sharing)

### Visual Example:

```

صورة 7×7 ← فلتر 3×3 ينزلق عليها

↓

Output: 5×5

```

**Formula للأبعاد:**

```

Output = (W - K + 2P) / S + 1

W = input size

K = filter size

P = padding

S = stride

```

---

## 3️⃣ المكونات الرئيسية للـ CNN

### A) Convolution Layer (slides 48-62)

| | |

|---|---|

| **Input** | C_in × H × W |

| **Filters** | C_out × C_in × K_h × K_w |

| **Output** | C_out × H' × W' |

| **Parameters** | C_out × C_in × K_h × K_w + C_out (bias) |

**مثال عملي (slide 88-93):**

```

Input: 3×32×32

10 filters: 5×5×3, stride=1, pad=2

Output: 10×32×32 ← (32+4-5)/1+1 = 32

Parameters: 10 × (3×5×5 + 1) = 10 × 76 = 760

Operations: 10×32×32 × 75 = 768,000 multiply-add

```

### B) Activation Function (ReLU)

```

ReLU(x) = max(0, x)

```

> بسيطة، سريعة، وبتمنع مشكلة vanishing gradient

### C) Pooling Layer (slides 99-103)

**الهدف:** تصغير الحجم المكاني (Downsampling)

**Max Pooling (الأكثر شيوعاً):**

```

Input: 4×4 Output: 2×2

┌──┬──┬──┬──┐ ┌──┬──┐

│1 │1 │2 │4 │ │6 │8 │

├──┼──┼──┼──┤ → ├──┼──┤

│5 │6 │7 │8 │ │3 │4 │

├──┼──┼──┼──┤ └──┴──┘

│3 │2 │1 │0 │

├──┼──┼──┼──┤

│1 │2 │3 │4 │

└──┴──┴──┴──┘

```

| | |

|---|---|

| **Common setting** | K=2, S=2 → 2× downsampling |

| **Parameters** | 0 (ما في أوزان!) |

| **Benefit** | Translation invariance (slide 104-106) |

### D) Fully Connected Layer (النهاية)

بعد طبقات Conv+Pool متعددة، الصورة بتصير صغيرة جداً. هون بنبسطها ونطبق Linear Classifier عادي.

---

## 4️⃣ Architecture كامل لـ CNN

```

Input: 32×32×3

↓

[CONV + ReLU] → 6 filters, 5×5, pad=0 → 28×28×6

↓

[POOL] → 2×2, stride=2 → 14×14×6

↓

[CONV + ReLU] → 10 filters, 5×5 → 10×10×10

↓

[POOL] → 2×2, stride=2 → 5×5×10

↓

[FC] → flatten + Linear → Class scores

```

---

## 5️⃣ Translation Equivariance (slide 104-106)

**هذا مفتاح CNN!**

```

إذا حركت الصورة يمين ← الـ Conv output بيتحرك يمين بنفس المقدار

Conv(Translate(X)) = Translate(Conv(X))

```

> **معنى عملي:** الفلتر نفسه بيكتشف "حافة" (edge) سواء كانت باليسار أو اليمين — ما بيحتاج يتعلمها مرتين!

---

## 6️⃣ شو بتتعلم الفلاتر؟ (slides 66-69)

| الطبقة | شو بتتعلم؟ |

|--------|-----------|

| **Layer 1** | حواف بسيطة (edges)، ألوان متعاكسة (slide 68) |

| **Layer 2** | أشكال بسيطة (textures, corners) |

| **Layer 6** | أجزاء معقدة (عيون، حروف، أذنين — slide 69) |

> **Hierarchical learning:** من بسيط لمعقد!

---

## 7️⃣ Hyperparameters المهمة

### Convolution:

| Parameter | Common Values | Effect |

|-----------|-------------|--------|

| K (filter size) | 3, 5, 1 | حجم الفلتر |

| S (stride) | 1, 2 | كم بتنزلق كل خطوة |

| P (padding) | (K-1)/2 | يحافظ على الحجم |

### Pooling:

| Parameter | Common Values |

|-----------|-------------|

| K (kernel) | 2 |

| S (stride) | 2 |

---

## 8️⃣ التاريخ والتطور (slides 33-41)

| السنة | الحدث |

|-------|-------|

| **1998** | LeNet (أول CNN ناجحة — Yann LeCun) |

| **2012** | AlexNet — فازت بـ ImageNet، بدأت ثورة الـ Deep Learning |

| **2012-2020** | CNNs سيطرت على كل مهام الرؤية (Detection, Segmentation, Captioning, Generation) |

| **2021+** | Transformers (ViT) بدأت تنافس CNNs |

---

## 9️⃣ ملخص CNN vs Fully Connected

| | Fully Connected | CNN |

|---|:---:|:---:|

| **Input** | Vector (flattened) | 3D Volume (H×W×C) |

| **Connectivity** | كل نورون متصل بكل المدخلات | Local connectivity (K×K) |

| **Parameters** | ضخمة | قليلة (parameter sharing) |

| **Spatial structure** | ❌ مضيعة | ✅ محفوظة |

| **Translation** | ❌ | ✅ Equivariant |

---

## 🔑 المعادلة الذهبية

```

CNN = Conv (استخراج features) + ReLU (تفعيل) + Pool (تصغير) + FC (تصنيف)

```

> **الهدف:** كل طبقة Conv بتستخرج features أعقد من السابقة، و Pooling بتصغر الحجم لحتى النورونات بالطبقات العميقة "تشوف" مناطق أكبر من الصورة (Receptive Field — slides 80-83).