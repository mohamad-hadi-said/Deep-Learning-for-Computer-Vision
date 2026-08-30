 
## 1. لماذا RNN لم تكن كافية؟

في المحاضرة السابقة، رأينا أن الـ **RNNs** تُعالج التسلسلات بشكل متكرر: كل خطوة تعتمد على الخطوة السابقة. لكن عند الترجمة الآلية (Seq2Seq)، كان هناك **عائق حرج**:

> **الـ Context Vector** هو vector واحد ثابت الطول (مثلاً 1024) يُلخّص جملة بأكملها.

لو كانت الجملة `"We see the sky"`، فـ 1024 رقم قد تكفي. لكن لو أردنا ترجمة **فقرة كاملة** أو **كتاب**؟ يصبح من المستحيل تلخيص كل هذا المعنى في vector واحد. هذا ما يُسمّيه المحاضر **"Communication Bottleneck"**.

---

## 2. آلية Attention بالتفصيل الدقيق

الحل: في كل خطوة من خطوات الـ Decoder، دعه ينظر **للجملة المصدرية كاملةً** ويختار ما يهمّه الآن.

### الخطوات الرياضية:

**أ) Encoder:** يُنتج سلسلة من الحالات المخفية:
$$h_1, h_2, h_3, h_4$$

**ب) Decoder Step t:** لدينا حالة مخفية $s_t$.

**ج) Alignment Scores:** نحسب درجة التشابه بين $s_t$ وكل $h_i$:
$$\text{score}_i = f(s_t, h_i)$$

حيث $f$ في البداية كانت طبقة Linear: $f(s_t, h_i) = W \cdot [s_t; h_i]$

**د) Softmax:** نحوّل الـ Scores لتوزيع احتمالي:
$$a_i = \frac{e^{\text{score}_i}}{\sum_j e^{\text{score}_j}}$$

**هـ) Context Vector:** متوسط مرجح:
$$C_t = \sum_{i=1}^{4} a_i \cdot h_i$$

**و) Decoder Update:** نُدخل $C_t$ للـ Decoder مع الـ Token السابق.

> **الجمال:** هذه العملية كلها **Differentiable**. لا نُخبر الشبكة "انظر لهذه الكلمة" — هي تتعلم ذلك بنفسها عبر الـ Loss Function.

### مثال الترجمة (English → French):
عندما يُنتج الـ Decoder كلمة `"l'accord"`، نجد أن الـ Attention Weights تُركّز بشكل حاد على كلمة `"agreement"` في الإنجليزية. وعند `"août"` تركّز على `"August"`. لكن العجيب هو أنها تكتشف **الترتيب المختلف** للكلمات تلقائياً (مثل `"European Economic Area"` الذي يأتي بترتيب مختلف بالفرنسية)!

---

## 3. من Attention إلى Operator عام

هنا يأتي التحوّل الفكري الأهم في المحاضرة: **لنُفكّك Attention عن RNN**.

### Query, Key, Value — التشبيه العميق:

| المفهوم | التشبيه العملي | الدور |
|---------|---------------|-------|
| **Query** | `"ما هي أفضل جامعة؟"` | ما نبحث عنه الآن |
| **Key** | عناوين/وسوم الصفحات | ما نُطابق الـ Query ضده |
| **Value** | محتوى الصفحة: `"Stanford"` | المعلومة الفعلية المُسترجعة |

> لماذا نفصل Key عن Value؟ لأن ما تبحث عنه **ليس بالضرورة** ما تريد استرجاعه!

### Scaled Dot-Product Attention:

المعادلات التي يُبنى عليها كل شيء:

$$Q = X \cdot W_q, \quad K = X \cdot W_k, \quad V = X \cdot W_v$$

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

**لماذا القسمة على $\sqrt{d_k}$؟**
عندما يكون الـ Dimension كبيراً (512 أو 1024)، قيم الـ Dot Product تنمو بشكل كبير. هذا يُدخل الـ Softmax في منطقة **التشبع (Saturation)** حيث التدرجات $\approx 0$. القسمة على $\sqrt{d_k}$ تُبقي القيم في نطاق "صحي" للـ Softmax.

---

## 4. Self-Attention — الثورة الحقيقية

**Cross Attention:** Query من مصدر، Key/Value من مصدر آخر (مثل الترجمة).

**Self-Attention:** كل الـ Q, K, V تُشتق من **نفس الإدخال**! كل vector ينظر لباقي الـ vectors ويتحدث معهم.

### خاصية Permutation Equivariance:
إذا بدّلت ترتيب الإدخالات (shuffled)، ستحصل على نفس الإخراجات ولكن بترتيب مبدّل. هذا يعني أن Self-Attention يعامل البيانات كـ **Set** وليس كـ **Sequence**.

**المشكلة:** في اللغة، الترتيب مهم جداً! `"الرجل أكل التفاحة"` ≠ `"التفاحة أكلت الرجل"`

**الحل:** **Positional Embeddings** — نضيف لكل vector معلومات عن موقعه في التسلسل.

---

## 5. Masked Self-Attention

في توليد النص (Language Modeling)، لا يجوز للكلمة النظر للمستقبل:

```
الإدخال:  Attention is very cool
الموضع 3 (very): يجب أن يرى فقط [Attention, is, very]
الموضع 4 (cool): يرى [Attention, is, very, cool]
```

**التنفيذ:** نضع **$-\infty$** في الـ Scores Matrix للمواضع المستقبلية قبل الـ Softmax. بعد الـ Softmax، تصبح هذه المواضع = 0.

---

## 6. Multi-Head Attention

بدلاً من تشغيل Attention مرة واحدة، نشغله **H مرات** (مثلاً 8 أو 16) بـ Weights مختلفة تماماً.

- **Head 1:** يركز على العلاقات النحوية (Subject → Verb)
- **Head 2:** يركز على المعنى الدلالي
- **Head 3:** يركز على المسافات المكانية (للصور)

الإخراجات تُلصق (Concatenate) ثم تُمر عبر Linear Projection واحدة.

> **في الممارسة:** يُحسب Multi-Head Attention بكفاءة باستخدام **One Big Batched Matrix Multiply** — هذا يجعله سريعاً جداً على GPUs.

---

## 7. Transformer Block — البنية الكاملة

```
Input: X
    ↓
Multi-Head Self-Attention
    ↓
+ Residual Connection (X + Attention(X))
    ↓
Layer Normalization
    ↓
Feed-Forward Network (FFN) ← طبقتان، تُطبق على كل vector بشكل مستقل
    ↓
+ Residual Connection
    ↓
Layer Normalization
    ↓
Output
```

**لماذا نحتاج FFN بعد Attention؟**
- **Attention:** "لنتحدث جميعاً ونرى ما يقوله الآخرون" (توزيع المعلومات)
- **FFN:** "الآن لكل واحد منا مساحته الخاصة للتفكير" (معالجة مستقلة)

**الـ Transformer الكامل = تكديس (Stack) لـ N Blocks.** الورقة الأصلية (2017) استخدمت 12 Block و~200M parameter. اليوم نرى مئات الـ Blocks وترليونات الـ Parameters!

---

## 8. لماذا Transformers سادت العالم؟

| الخاصية | RNN | CNN | Self-Attention |
|---------|-----|-----|----------------|
| **التوازي** | ❌ تسلسلي | ✅ موازي | ✅✅ موازي جداً |
| **Receptive Field** | كامل (بطيء) | محلي | **كامل في طبقة واحدة!** |
| **قابلية التوسع** | صعبة | جيدة | **ممتازة** (Matrix Mults) |
| **التعقيد** | O(n) | O(n) | O(n²) — لكن GPU-friendly |

> المحاضر يسمّيها: **"Attention is all you need"** — لأنك تستطيع الذهاب بعيداً باستخدام Attention فقط!

---

## 9. Vision Transformers (ViT)

نفس البنية تُستخدم للصور:
1. نقسم الصورة إلى **Patches** (16×16 بكسل).
2. كل Patch يُسقط إلى Vector.
3. نضيف Positional Embeddings.
4. نُدخلها لـ Transformer Blocks.
5. نستخدم **[CLS] Token** أو Average Pooling للتصنيف.

---

## 10. الخلاصة الذهبية

> **"Transformers are basically the architecture that we use for almost all problems in deep learning today."**

المحاضرة تُعلّمك ليس فقط "كيف يعمل Transformer"، بل **"لماذا ولد من RNNs"** و**"ما المشكلة التي حلها"**. هذا الفهم العميق هو ما يُميّك كمهندس تعلم عميق.