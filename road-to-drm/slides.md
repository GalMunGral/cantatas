---
theme: default
layout: center
highlighter: shiki
---

# 通往DRM之路: 用户作为敌手
The Road to DRM: User as Adversary

---
layout: center
---
```html
<video src="sample.mp4" constrols></video>
```
---
layout: center
---

# 可以通过用户界面下载

---
layout: center
---
```html
<video
  src="sample.mp4"
  constrols
  controlslist="nodownload" 
  oncontextmenu="event.preventDefault()"
></video>
```
---
layout: center
---

# 可以通过控制台找到资源下载

---
layout: center
---

# Media Source Extensions (MSE)

```js
window.onload = async () => {
  const video = document.querySelector("video");
  const source = new MediaSource();
  source.onsourceopen = async () => {
    const sourceBuf = source.addSourceBuffer(
      'video/mp4; codecs="avc1.4D4028, mp4a.40.2"'
    );
    const res = await fetch("log.txt");
    const buf = await res.arrayBuffer();
    sourceBuf.appendBuffer(buf);
  };
  video.src = URL.createObjectURL(source);
};
```

---
layout: center
---

# [ISO BMFF](https://www.w3.org/TR/mse-byte-stream-format-isobmff/)
- 1 Initialization Segment
  - 1 File Type (`ftyp`) box
  - 1 Movie (`moov`) box
    - 1 Movie Extends (`mvex`) box - 表明后面有 `moof` box
- 1+ Media Segment
  - (optional) Segment Type (`styp`) box
  - 1 Movie Fragment (`moof`) box
  - 1+ Media Data (`mdat`) boxes

---
layout: center
---

```bash
brew install mp4box

# 不生成 segment 文件
mp4box -dash 3000 input.mp4

# 生成 segment 文件
mp4box -dash 3000 \
  -segment-name 'error-log-$Number$' \
  -segment-ext txt \
  -init-segment-ext txt \
  -out error-report.mpd \
  input.mp4
```

---
layout: center
---

# 可以下载所有片段然后拼接
`cat seg1 seg2 seg3 > out.mp4`

---
layout: center
---
```bash
openssl enc -aes-256-cbc -pass pass:"<password>" -in <input-file> -base64 -out <ouput-file>
```
(AES-256-CBC 和 base64 编码是为了便于对接 CyptoJS 库)

![cbc](/assets/cbc.png)

---
layout: center
---
# 可以解密

1. 如果用 K 加密视频, 那么要如何秘密地获取 K? 用 K' 加密 K?
2. 那么要如何秘密地获取 K'? 再用 K'' 加密 K'?
3. 那么要如何秘密地获取 K''?

---
layout: center
---

# Diffie-Hellman 密钥交换

完全没有对方任何预先信息的条件下通过不安全信道创建起一个密钥

- $p$ 是一个公开的很大的素数, $1 < g < p$
- A 生成一个随机数 $m$ 然后把 $g^m \bmod p$ 给 B
- B 生成一个随机数 $n$ 然后把 $g^n \bmod p$ 给 A
- 双方可以得到密钥
$$
(g^m \bmod p) ^ n \bmod p = (g^n \bmod p) ^ m \bmod p = g^{mn} \bmod p
$$
- 没有高效的算法可以从 $g^m \bmod p$ 算出 $m$ (离散对数), 只能得到
$$
(g^m \bmod p) \cdot (g ^ n \bmod p) \bmod p = g^{m + n} \bmod p
$$

---
layout: center
---

# 可以打断点

---
layout: center
---

```js
function DEVTOOL_TRAP() {
  while (IS_SOMEONE_WATCH__in__G()) {}

  function IS_SOMEONE_WATCH__in__G() {
    const HUMAN_CANT_BE_THAT_FAST = 10;
    let t1 = performance.now();
    debugger;
    let t2 = performance.now();
    return t2 - t1 > HUMAN_CANT_BE_THAT_FAST;
  }
}
```

---
layout: center
---

```js
(function trap() { DEVTOOL_TRAP(); setTimeout(trap); })()
```

---
layout: center
---
# 问题
想象一下浏览器的任务队列
- $T$ 表示任务, $S$ 表示禁止打断点的任务
- $X$ 表示 `DEVTOOL_TRAP` 的执行
- $|$ 表示打开控制台的动作 (代码执行中是无法打开控制台的)
```bash
X T X T|X S X T X T
```
```bash
X T X T X|S X T X T
```
- 我们需要 X 和 S 同步执行

---
layout: center
---

```js
async function diffieHellman() {
  const M = BigInt(Math.floor(Math.random() * 0xabcdef));
  const GMP = modExp(DH.G, M, DH.P);
  const res = await fetch(`key-exchange?gmp=${GMP}`);
  const GNP = BigInt(await res.text());
  const GMNP = btoa(modExp(GNP, M, DH.P));
  return GMNP;
}
```

---
layout: center
---

```js
let __in__, __out__;

async function diffieHellman() {
  const M = BigInt(Math.floor(Math.random() * 0xabcdef));
  const GMP = modExp(DH.G, M, DH.P);
  __out__ = fetch(`key-exchange?gmp=${GMP}`);

  /********************************************/ __in__ = await __out__;

  const res = __in__
  __out__ = res.text();

  /********************************************/ __in__ = await __out__;

  const GNP = BigInt(__in__);
  const GMNP = btoa(modExp(GNP, M, DH.P));
  return GMNP;
}
```

---
layout: center
---

```js
let __in__, __out__;

async function diffieHellman() {
  DEVTOOL_TRAP();
  const M = BigInt(Math.floor(Math.random() * 0xabcdef));
  const GMP = modExp(DH.G, M, DH.P);
  __out__ = fetch(`key-exchange?gmp=${GMP}`);

  /********************************************/ __in__ = await __out__;

  DEVTOOL_TRAP()
  const res = __in__
  __out__ = res.text();

  /********************************************/ __in__ = await __out__;

  DEVTOOL_TRAP()
  const GNP = BigInt(__in__);
  const GMNP = btoa(modExp(GNP, M, DH.P));
  return GMNP;
}
```

---
layout: center
---
```js
const diffieHellman = secure(function* () {
  const M = BigInt(Math.floor(Math.random() * 0xabcdef));
  const GMP = modExp(DH.G, M, DH.P);
  const res = yield fetch(`key-exchange?gmp=${GMP}`);
  const GNP = BigInt(yield res.text());
  const GMNP = btoa(modExp(GNP, M, DH.P));
  return GMNP;
});
```

---
layout: center
---

```js
function secure(gen) {
  return (...args) =>
    new Promise((resolve) => {
      runAsync(gen(...args), null, resolve);
    });

  function runAsync(gen, arg, resolve) {
    DEVTOOL_TRAP();
    const res = gen.next(arg);
    if (res.done) {
      resolve(res.value);
    } else {
      if (res.value instanceof Promise) {
        res.value.then((val) => runAsync(gen, val, resolve));
      }
    }
  }
}
```