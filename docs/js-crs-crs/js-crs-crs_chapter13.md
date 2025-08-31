

## 第五章：13 写作 一首 歌曲



![](img/opener.png)

现在你已经学到了足够的 Tone.js 基础和声音合成的知识，能够写出一首简单的歌曲。我们的歌曲将由几个乐器组成：上一章中开发的鼓，喇叭采样器，两个不同的合成贝斯部分，以及另一个合成器上演奏的和弦。

### 组织结构

我们的歌曲将重用上一章中的许多代码，但我们会对其进行重组，使得跟踪歌曲的构建过程更加容易。*index.html* 文件将与第十二章中的完全相同，但我们将从头开始创建一个新的 *script.js* 文件，并将其组织成四个逻辑部分：

**乐器**    用于实例化和设置乐器

**序列化**    用于创建循环播放的音符序列

**歌曲**    用于安排每个序列的开始和结束

**事件处理**    处理启动歌曲播放的点击事件的代码

我们将用多行注释来标记这四个部分，以便更容易导航 *script.js* 文件。清单 13-1 显示了这些注释的样子。你现在可以按照这个顺序将它们添加到文件中。

```
/////////////////
// Instruments //
/////////////////

////////////////
// Sequencing //
////////////////

//////////
// Song //
//////////

////////////////////
// Event Handling //
//////////////////// 
```

清单 13-1：标明 script.js的主要部分的注释

在本章中，当我们构建歌曲时，我会告诉你将每个新代码片段添加到特定部分的末尾。这些注释将帮助你快速找到新代码应放置的位置。

### 事件处理

让我们从编写 *script.js* 中的事件处理部分开始。这段代码几乎与我们在上一章开头编写的代码相同：它创建了一个点击事件监听器，在用户点击按钮时切换播放按钮和“正在播放”段落的样式，并调用必要的 Tone.js 方法来开始播放歌曲。将清单 13-2 中的内容输入到代码的事件处理部分。

```
`--snip--`
////////////////////
// Event Handling //
////////////////////

let play = document.querySelector("#play");
let playing = document.querySelector("#playing");

play.addEventListener("click", () => {
  // Hide this button
  play.style = "display: none";
  playing.style = " ";

 Tone.start();

  // Modify this to start playback at a different part of the song
❶ Tone.Transport.position = "0:0:0";
  Tone.Transport.start();
}); 
```

清单 13-2：事件处理代码

与[Feynman 学习方法](https://wiki.example.org/feynmans_learning_method)相比，这段代码的一个重要区别是，我们使用 Tone.Transport.position 在调用 Tone.Transport.start ❶之前设置传输的起始位置。在这里，我们将起始位置设置为"0:0:0"，这是默认值，因此这行代码严格来说并不是必须的。然而，包含这行代码可以方便你在添加新元素时修改起始位置，这样就不必每次都听完整首歌曲。例如，如果你想跳过前 20 小节，你可以将 Tone.Transport.position 的值更改为"20:0:0"。

与上一章不同，所有创建乐器和序列的代码都放在事件处理程序之外。所有这些代码都可以在用户按下播放按钮之前执行。只有 Tone.start 调用必须放在事件处理程序内，歌曲才能正常工作。如果我们愿意，甚至可以将 Tone.Transport 的相关代码移到事件处理程序之外，但在 Tone.start 之后执行这些代码更自然。

### 制作鼓点

现在，让我们创建伴随歌曲的鼓点。我们将使用上一章中创建的相同的踩镲、小军鼓和低音鼓声。首先，我们将声明这些乐器，如 Listing 13-3 所示。将这段代码添加到*script.js*的乐器部分。

```
/////////////////
// Instruments //
/////////////////

❶ function mkDrums() {
  let reverb = new Tone.Reverb({
    decay: 1,
    wet: 0.3
  }).toDestination();

  let hiHatFilter = new Tone.Filter(15000, "bandpass").connect(reverb);

  let hiHat = new Tone.NoiseSynth({
    envelope: {
      attack: 0.001, decay: 0.1, sustain: 0, release: 0
    },
    volume: -6
  }).connect(hiHatFilter);

 class Snare {
    constructor() {
      this.noiseFilter = new Tone.Filter(5000, "bandpass").connect(reverb);
      this.noiseSynth = new Tone.NoiseSynth({
        envelope: {
          attack: 0.001, decay: 0.1, sustain: 0, release: 0
        },
        volume: -12
      }).connect(this.noiseFilter);

      this.synth = new Tone.Synth({
        envelope: {
          attack: 0.0001, decay: 0.1, sustain: 0, release: 0
        },
        oscillator: {type: "sine"},
        volume: -12
      }).connect(reverb);
    }

    triggerAttackRelease(duration, when) {
      this.noiseSynth.triggerAttackRelease(duration, when);
      this.synth.triggerAttackRelease("G3", duration, when);
    }
  }

  let snare = new Snare();

  let kick = new Tone.MembraneSynth({
    pitchDecay: 0.02,
    octaves: 6,
    volume: -9
  }).connect(reverb);

❷ return {hiHat, snare, kick};
}

let drums = mkDrums();
`--snip--` 
```

Listing 13-3: 声明鼓组

这段代码与我们在上一章编写的代码相同，但为了保持代码的组织性，我将所有的鼓组设置代码，包括混响效果，移动到一个名为 mkDrums（“制作鼓组”）的单独函数中❶。这个函数返回一个包含三种鼓的对象❷。我们使用了一种新的语法来创建这个对象，叫做*对象字面量简写语法*。使用这种简写语法，我们不需要输入{hiHat: hiHat, snare: snare, kick: kick}，而只需输入{hiHat, snare, kick}。这种写法只有在属性名与变量名相同的情况下才有效。

现在我们已经声明了鼓组，接下来我们将创建实际的鼓点模式。我们将使用在上一章中开发的相同的单小节模式，每个八分音符上都有踩镲声，低音鼓和小军鼓声每个四分音符交替出现。将 Listing 13-4 添加到代码的序列部分。

```
`--snip--`
////////////////
// Sequencing //
////////////////

// Converts a string to an array of notes or nulls.
// Dots in the string become nulls in the array and are silent.
❶ function mkSequence(pattern) {
  return pattern.split(" ").map(value => {
    if (value == ".") {
      return null;
    } else {
      return value;
    }
  });
}

❷ let drumPattern = {
  kick:  "x…x…",
  snare: "..x…x.",
  hiHat: "xxxxxxxx",
};

let hiHatSequence = new Tone.Sequence(time => {
  drums.hiHat.triggerAttackRelease("16n", time);
}, mkSequence(drumPattern.hiHat), "8n");

let snareSequence = new Tone.Sequence(time => {
  drums.snare.triggerAttackRelease("16n", time);
}, mkSequence(drumPattern.snare), "8n");

let kickSequence = new Tone.Sequence(time => {
    drums.kick.triggerAttackRelease(50, "16n", time);
}, mkSequence(drumPattern.kick), "8n");
`--snip--` 
```

Listing 13-4: 鼓点序列

再次强调，这段代码与我们在第十二章中编写的代码相同。我们从一个辅助函数 mkSequence ❶开始，它接收一个由 x 和点组成的模式，并将其转换为 Tone.Sequence 可以使用的音符信息。然后，我们将想要的模式存储在 drumPattern 对象❷中，并使用 Tone.Sequence 为每个乐器生成序列。

创建鼓点所剩下的工作就是安排序列循环播放，持续大部分歌曲时长，如列表 13-5 所示。将这段代码添加到 *script.js* 文件的 Song 部分。

```
`--snip--`
//////////
// Song //
//////////

hiHatSequence.start("0:0:0").stop("44:0:0");
snareSequence.start("0:0:0").stop("44:0:0");
kickSequence.start("0:0:0").stop("44:0:0");
`--snip--` 
```

列表 13-5：安排鼓点序列

在这里，我们告诉鼓在歌曲的开始处启动并持续演奏 44 小节。加载页面并点击 **Play**，你应该会听到和之前一样的鼓声，但会持续更长时间。当你听腻了时，可以重新加载页面停止鼓声的播放。

### 添加低音线

接下来，我们将添加几个低音合成器，并让它们演奏两个独立的低音线。首先，我们通过将 列表 13-6 中的代码添加到 Instruments 部分的末尾（紧接着 Sequencing 部分之前）来创建这些合成器。

```
`--snip--`
let lowBass = new Tone.FMSynth({
  oscillator: {
  ❶ type: "triangle"
  },
  envelope: {
    attack: 0.0001, decay: 0.5, sustain: 0.3, release: 0.1
  },
  volume: -3
}).toDestination();

let highBass = new Tone.FMSynth({
  oscillator: {
  ❷ type: "square"
  },
  envelope: {
    attack: 0.0001, decay: 0.1, sustain: 0.3, release: 0.1
  },
  volume: -9
}).toDestination();
`--snip--` 
```

列表 13-6：创建低音乐器

在这里，我们声明了两个低音乐器，分别叫做 lowBass 和 highBass。它们都使用了一种我们尚未见过的合成器，叫做 FMSynth。*FM* 是 *频率调制*（Frequency Modulation）的缩写，*FM 合成* 涉及使用一个振荡器来调制或修改另一个振荡器的频率。这种合成方式比单一振荡器产生的声音更丰富，也非常适合作为低音合成器。Tone.FMSynth 中有很多可以调整的参数（例如，调制的程度、两个振荡器之间的频率关系、振荡器的波形等等），但我们将主要使用默认值。我们所做的仅仅是设置振荡器类型（"triangle" 为 "lowBass" ❶ 和 "square" 为 highBass ❷），以及包络和音量。

为了生成低音序列，我们将采用一种与当前的 mkSequence 辅助函数略有不同的技术。这个辅助函数非常适合用于像鼓这样的场景，在那里你只需要一个字符来决定一个音符是否被演奏，但它不适用于低音线，我们需要提供音符名称，而这些名称至少有两个字符（如 C3 或 F#4）。我们可能选择的一种表示序列的方式可以是这样的：

```
"C3|  |  |C3|  |  |G2|B2"
```

垂直的管道字符用于分隔，每对管道之间是我们想要演奏的音符或空格，空格表示沉默。（这里写出的序列是 Ben E. King 的《Stand by Me》中的低音线的开头。）

列表 13-7 给出了 mkPipeSequence 的定义，我们将用它来为低音线编排序列。它接收像《Stand by Me》中的字符串，并将其转换成音符名称和空值的数组。将此函数插入到 *script.js* 的 Sequencing 部分，紧接着 mkSequence 的定义。

```
`--snip--`
// Converts a string to an array of notes or nulls.
// Spaces between pipes in the string become nulls in the array and are silent.
function mkPipeSequence(pattern) {
  return pattern.split("|").map(value => {
  ❶ if (value.trim() == " ") {
      return null;
    } else {
      return value;
    }
  });
}
`--snip--` 
```

列表 13-7：mkPipeSequence 函数

这个函数使用 split("|")通过管道符分割字符串。以“Stand by Me”为例，这将返回数组["C3", " ", " ", "C3", " ", " ", "G2", "B2"]。然后我们对这些值进行映射。trim 方法❶会去除字符串两端的任何空白，因此" ".trim()会返回""，一个空字符串。我们将返回数组中的所有空字符串替换为 null，并将音符名称原样传递，最终返回值为["C3", null, null, "C3", null, null, "G2", "B2"]。

接下来，我们要为两个低音线创建实际的序列（这里我们不会借用“Stand by Me”）。将清单 13-8 中的代码添加到 Sequencing 部分的末尾。

```
`--snip--`
let lowBassSequence = new Tone.Sequence((time, note) => {
  lowBass.triggerAttackRelease(note, "16n", time, 0.6);
}, mkPipeSequence("G2|  |  |G2|G2|  |  |  "), "8n");

let highBassSequence = new Tone.Sequence((time, note) => {
  highBass.triggerAttackRelease(note, "16n", time, 0.3);
}, mkPipeSequence("G3|F3|E3|D3|G2|D3|G3|D3"), "8n");
`--snip--` 
```

清单 13-8：低音序列

这里有两个低音部分：低音部分每小节播放三个八分音符，而高音部分则连续播放八分音符。

最后，我们需要将这些序列与传输进行调度，如清单 13-9 所示。此代码应添加到 Song 部分的末尾。

```
`--snip--`
lowBassSequence.start("0:0:0").stop("47:3:0");
highBassSequence.start("4:0:0").stop("47:3:0");
`--snip--` 
```

清单 13-9：调度低音序列

低音序列从一开始就启动，高音序列在四小节后开始。两者都将继续循环，直到 48 小节中途为止。这样，低音部分将在鼓声停止后继续几小节。

如果你现在刷新页面并点击播放，你将听到歌曲的开头！我们不仅有鼓和低音，还有一些非常基础的结构，第二个低音部分在四小节后加入，而鼓声在低音之前结束。当前歌曲中，最后的低音独奏是最具戏剧性的部分。要单独听这一部分，你可以在代码的事件处理部分修改 Tone.Transport.Position 的值。如果将其设置为“40:0:0”并重新加载，你将跳到歌曲的最后八小节。

### 添加和弦

接下来，我们将为歌曲添加一些和弦。这首歌将有两个独立的和弦序列，我们将为歌曲中的不同时间安排它们，以增加结构性和多样性。

首先，我们需要创建播放和弦的乐器。相关代码在清单 13-10 中；将其插入到乐器部分的末尾。

```
`--snip--`
let chordSynth = new Tone.PolySynth(Tone.Synth, {
  oscillator: {
    type: "triangle"
  },
  volume: -12
}).toDestination();
`--snip--` 
```

清单 13-10：和弦合成器

我们需要一个 PolySynth，因为该乐器将同时播放多个音符（这就是和弦）。PolySynth 基于常规的 Synth，使用默认的振幅包络和三角波振荡器。

接下来，我们将为和弦创建编排代码。与其每次想要在序列中播放一个和弦时手动写出它，我们将创建一些命名的和弦，然后使用这些和弦名称创建序列。将列表 13-11 中的代码插入到编排部分的末尾。

```
`--snip--`
❶ let chords = {
  1: ["D4", "G4"],
  2: ["E4", "G4"],
  3: ["C4", "E4", "G4"],
  4: ["B3", "F4", "G4"],
};

❷ function playChord(time, chordName) {
❸ let notes = chords[chordName];
  chordSynth.triggerAttackRelease(notes, "16n", time, 0.6);
}

❹ let chordSequence1 = new Tone.Sequence((time, chordName) => {
  playChord(time, chordName);
}, mkSequence("1…2…3..4…31…2…3..4.343"), "8n");

❺ let chordSequence2 = new Tone.Sequence((time, chordName) => {
  playChord(time, chordName);
}, mkSequence("3…2…4..1.213"), "8n"); 
`--snip--` 
```

列表 13-11：和弦编排

我们首先创建一个名为 chords 的对象，包含我们将要编排的四个和弦 ❶。我们可以给它们任何名字，但为了简化，我使用数字 1、2、3 和 4 来表示这些和弦（不过请注意，由于这些是对象的键，数字会被当作字符串处理）。每个和弦编号对应一个音符名称的数组，这是 PolySynth 所需的格式。这两个和弦序列只是这四个和弦的不同排列。

接下来是一个用于播放和弦的辅助函数 ❷。这个 playChord 函数接受播放和弦的时间和和弦的名称（作为字符串，取值为 1 到 4 中的一个）。然后，它在 chords 对象中查找并提取由给定和弦名称键入的音符数组 ❸。函数的最后会调用 triggerAttackRelease 方法来触发和弦合成器，传入音符名称的数组。由于它是 PolySynth，我们的和弦合成器可以同时演奏和弦中的所有音符。

最后，我们创建了两个序列，分别叫做 chordSequence1 ❹和 chordSequence2 ❺。这两个序列的回调函数都是我们的 playChord 函数。我们还使用了之前用于编排鼓点的 mkSequence 辅助函数，但在这里，字符串中的值要么是点（静默），要么是和弦名称。与低音线不同，mkSequence 在这里可以正常工作，因为每个和弦名称都是一个单独的字符，我们有 playChord 函数来将和弦名称重新解释为音高。与鼓点一样，我们将"8n"作为最后一个参数传递给 Tone.Sequence，这意味着每个点或和弦名称代表一个八分音符。第一个序列有 32 个八分音符长，或者 4 小节。第二个序列有 16 个八分音符长，或者 2 小节。

现在我们将真正安排这些序列与时间轨道的同步。将列表 13-12 中的代码添加到 Song 部分的末尾。

```
`--snip--`
chordSequence1.start("8:0:0").stop("24:0:0");
chordSequence2.start("24:0:0").stop("32:0:0");
chordSequence1.start("32:0:0").stop("40:0:0");
`--snip--` 
```

列表 13-12：安排和弦序列

第一个序列在 8 小节后开始播放，并持续播放到第 24 小节结束，这时已经播放了 16 小节，或者说是第一个序列的四个完整循环。接下来，第二个序列接管，持续播放到第 32 小节；这时播放了 8 小节，或者是第二个序列的四个完整循环。最后，第一个序列重新回归，播放到第 40 小节；这也是 8 小节，或者是第一个序列的两个完整循环。

尝试刷新浏览器并再次聆听歌曲。确保在事件处理程序中将 Tone.Transport.position 设置为 "0:0:0" 来从头开始播放。如果你不想等八小节才能进入和弦，设置为 "8:0:0" 就能从和弦开始播放。

### 播放旋律

现在我们已经有了鼓、贝斯和和弦，我们的歌曲唯一缺少的就是旋律。我们将使用在上一章创建的小号采样器，并使用 Tone.Part 来安排音符，通过它我们可以轻松地分别安排旋律中每个音符的时机。

首先，我们会创建采样器，就像在第十二章中做的那样。将列表 13-13 中的代码添加到乐器部分的末尾。

```
`--snip--`
// Samples from freesound.org:
// https://freesound.org/people/MTG/sounds/357432/
// https://freesound.org/people/MTG/sounds/357336/
// https://freesound.org/people/MTG/sounds/357546/
let sampler = new Tone.Sampler({
  urls: {
    "C5": "trumpet-c5.mp3", 
    "D5": "trumpet-d5.mp3", 
    "F5": "trumpet-f5.mp3" 
  },
  baseUrl: "https://skilldrick-jscc.s3.us-west-2.amazonaws.com/",
  attack: 0,
  release: 1,
  volume: -24
}).toDestination();
`--snip--` 
```

列表 13-13：声明小号采样器

在这里，我们创建了一个 Tone.Sampler 乐器，使用与上一章相同的三个样本。不过，请注意，我们不再使用采样器的 onload 属性来告诉它在样本下载完成后应该做什么。这有点偷懒，但我知道小号不会在歌曲开始时演奏，我依赖于在它们进入时样本已经下载完成。正确的做法是隐藏播放按钮，直到样本下载完成，但那样会增加项目的复杂性。

列表 13-14 展示了为旋律安排音符的代码。将此代码添加到安排部分的末尾。

```
`--snip--`
let trumpetPart = new Tone.Part((time, note) => {
  sampler.triggerAttackRelease(note, "1n", time);
}, [
  ["0:0:0", "G5"],
  ["0:2:0", "C5"],
  ["1:0:0", "G5"],

  ["2:0:0", "D5"],
  ["2:2:0", "C5"],
  ["3:0:0", "B4"],

  ["4:0:0", "G5"],
  ["4:2:0", "C5"],
  ["5:0:0", "G5"],

  ["6:0:0", "D5"],
  ["6:2:0", "C5"],
  ["7:0:0", "B4"],
  ["7:2:0", "D5"],

  ["8:0:0", "C5"],
  ["8:2:0", "E5"],
 ["9:0:0", "F5"],
  ["9:2:0", "D5"],

  ["10:0:0", "C5"],
  ["10:2:0", "E5"],
  ["11:0:0", "D5"],

  ["12:0:0", "C5"],
  ["12:2:0", "E5"],
  ["13:0:0", "F5"],
  ["13:2:0", "D5"],

  ["14:0:0", "C5"],
  ["14:2:0", "E5"],
  ["15:0:0", ["B4", "G5"]]
]);
`--snip--` 
```

列表 13-14：旋律音符安排

提醒一下，Tone.Part 构造函数接受两个参数：一个用于播放每个时间/音符对的回调函数和一个时间/音符对的列表。在这里，回调函数会为每个时间/音符对在小号采样器上播放一个长音符（"1n"，即一个完整的小节）。第一个音符在 "0:0:0" 播放，第二个音符则在两拍后播放，即 "0:2:0"。由于音符大约四拍长，它们会重叠——我故意这么做，以增加旋律的趣味性。

这首曲子还不会播放，因为我们还没有指定*何时*播放。即使每个音符都有一个时间，这些时间是相对于部分开始安排的时间而言的。为了安排这个部分，我们只需在歌曲部分的末尾添加一些代码，如列表 13-15 所示。

```
`--snip--`
trumpetPart.start("16:0:0");
`--snip--` 
```

列表 13-15：安排小号部分

与我们到目前为止安排的序列不同，这一部分没有循环，因此不需要停止时间。我们告诉 Tone.js 在 16 小节后开始小号部分，这意味着该部分中的所有时间都相对于"16:0:0"。我们可以将两个时间加在一起，得到每个音符的实际安排时间（例如，"4:2:0" + "16:0:0" 就是 "20:2:0"）。

现在你可以听完整首歌曲了！在刷新页面之前，别忘了将 Tone.Transport.position 重置为 "0:0:0"。

### 完整代码

我们已经在文件中各个地方添加了代码，所以如果你把某些内容弄混了，或者只是想看看最终效果，清单 13-16 给出了*script.js*的完整内容。

```
/////////////////
// Instruments //
/////////////////

function mkDrums() {
  let reverb = new Tone.Reverb({
    decay: 1,
    wet: 0.3
  }).toDestination();

  let hiHatFilter = new Tone.Filter(15000, "bandpass").connect(reverb);

  let hiHat = new Tone.NoiseSynth({
    envelope: {
      attack: 0.001, decay: 0.1, sustain: 0, release: 0
    },
    volume: -6
  }).connect(hiHatFilter);

  class Snare {
    constructor() {
      this.noiseFilter = new Tone.Filter(5000, "bandpass").connect(reverb);
      this.noiseSynth = new Tone.NoiseSynth({
        envelope: {
          attack: 0.001, decay: 0.1, sustain: 0, release: 0
        },
        volume: -12
 }).connect(this.noiseFilter);

      this.synth = new Tone.Synth({
        envelope: {
          attack: 0.0001, decay: 0.1, sustain: 0, release: 0
        },
        oscillator: {type: "sine"},
        volume: -12
      }).connect(reverb);
    }

    triggerAttackRelease(duration, when) {
      this.noiseSynth.triggerAttackRelease(duration, when);
      this.synth.triggerAttackRelease("G3", duration, when);
    }
  }

  let snare = new Snare();

  let kick = new Tone.MembraneSynth({
    pitchDecay: 0.02,
    octaves: 6,
    volume: -9
  }).connect(reverb);

  return {hiHat, snare, kick};
}

let drums = mkDrums();

let lowBass = new Tone.FMSynth({
  oscillator: {
    type: "triangle"
  },
  envelope: {
    attack: 0.0001, decay: 0.5, sustain: 0.3, release: 0.1
  },
  volume: -3
}).toDestination();

let highBass = new Tone.FMSynth({
  oscillator: {
    type: "square"
  },
  envelope: {
    attack: 0.0001, decay: 0.1, sustain: 0.3, release: 0.1
  },
  volume: -9
}).toDestination();

let chordSynth = new Tone.PolySynth(Tone.Synth, {
  oscillator: {
    type: "triangle"
  },
 volume: -12
}).toDestination();

// Samples from freesound.org:
// https://freesound.org/people/MTG/sounds/357432/
// https://freesound.org/people/MTG/sounds/357336/
// https://freesound.org/people/MTG/sounds/357546/
let sampler = new Tone.Sampler({
  urls: {
    "C5": "trumpet-c5.mp3", 
    "D5": "trumpet-d5.mp3", 
    "F5": "trumpet-f5.mp3" 
  },
  baseUrl: "https://skilldrick-jscc.s3.us-west-2.amazonaws.com/",
  attack: 0,
  release: 1,
  volume: -24
}).toDestination();

////////////////
// Sequencing //
////////////////

// Converts a string to an array of notes or nulls.
// Dots in the string become nulls in the array and are silent.
function mkSequence(pattern) {
  return pattern.split(" ").map(value => {
    if (value == ".") {
      return null;
    } else {
      return value;
    }
  });
}

// Converts a string to an array of notes or nulls.
// Spaces between pipes in the string become nulls in the array and are silent.
function mkPipeSequence(pattern) {
  return pattern.split("|").map(value => {
    if (value.trim() == " ") {
      return null;
    } else {
      return value;
    }
  });
}

let drumPattern = {
  kick:  "x…x…",
  snare: "..x…x.",
  hiHat: "xxxxxxxx",
};

let hiHatSequence = new Tone.Sequence(time => {
  drums.hiHat.triggerAttackRelease("16n", time);
}, mkSequence(drumPattern.hiHat), "8n");

let snareSequence = new Tone.Sequence(time => {
  drums.snare.triggerAttackRelease("16n", time);
}, mkSequence(drumPattern.snare), "8n");

let kickSequence = new Tone.Sequence(time => {
  drums.kick.triggerAttackRelease(50, "16n", time);
}, mkSequence(drumPattern.kick), "8n");

let lowBassSequence = new Tone.Sequence((time, note) => {
  lowBass.triggerAttackRelease(note, "16n", time, 0.6);
}, mkPipeSequence("G2|  |  |G2|G2|  |  |  "), "8n");

let highBassSequence = new Tone.Sequence((time, note) => {
  highBass.triggerAttackRelease(note, "16n", time, 0.3);
}, mkPipeSequence("G3|F3|E3|D3|G2|D3|G3|D3"), "8n");

let chords = {
  1: ["D4", "G4"],
  2: ["E4", "G4"],
  3: ["C4", "E4", "G4"],
  4: ["B3", "F4", "G4"],
};

function playChord(time, chordName) {
  let notes = chords[chordName];
  chordSynth.triggerAttackRelease(notes, "16n", time, 0.6);
}

let chordSequence1 = new Tone.Sequence((time, chordName) => {
  playChord(time, chordName);
}, mkSequence("1…2…3..4…31…2…3..4.343"), "8n");

let chordSequence2 = new Tone.Sequence((time, chordName) => {
  playChord(time, chordName);
}, mkSequence("3…2…4..1.213"), "8n");

let trumpetPart = new Tone.Part((time, note) => {
  sampler.triggerAttackRelease(note, "1n", time);
}, [
  ["0:0:0", "G5"],
  ["0:2:0", "C5"],
  ["1:0:0", "G5"],

  ["2:0:0", "D5"],
  ["2:2:0", "C5"],
  ["3:0:0", "B4"],

  ["4:0:0", "G5"],
  ["4:2:0", "C5"],
  ["5:0:0", "G5"],

 ["6:0:0", "D5"],
  ["6:2:0", "C5"],
  ["7:0:0", "B4"],
  ["7:2:0", "D5"],

  ["8:0:0", "C5"],
  ["8:2:0", "E5"],
  ["9:0:0", "F5"],
  ["9:2:0", "D5"],

  ["10:0:0", "C5"],
  ["10:2:0", "E5"],
  ["11:0:0", "D5"],

  ["12:0:0", "C5"],
  ["12:2:0", "E5"],
  ["13:0:0", "F5"],
  ["13:2:0", "D5"],

  ["14:0:0", "C5"],
  ["14:2:0", "E5"],
  ["15:0:0", ["B4", "G5"]]
]);

//////////
// Song //
//////////

hiHatSequence.start("0:0:0").stop("44:0:0");
snareSequence.start("0:0:0").stop("44:0:0");
kickSequence.start("0:0:0").stop("44:0:0");

lowBassSequence.start("0:0:0").stop("47:3:0");
highBassSequence.start("4:0:0").stop("47:3:0");

chordSequence1.start("8:0:0").stop("24:0:0");
chordSequence2.start("24:0:0").stop("32:0:0");
chordSequence1.start("32:0:0").stop("40:0:0");

trumpetPart.start("16:0:0");

////////////////////
// Event Handling //
////////////////////

let play = document.querySelector("#play");
let playing = document.querySelector("#playing");

play.addEventListener("click", () => {
  // Hide this button
  play.style = "display: none";
  playing.style = " ";

  Tone.start();

 // Modify this to start playback at a different part of the song
  Tone.Transport.position = "0:0:0";
  Tone.Transport.start();
}); 
```

清单 13-16：完整代码

### 总结

在这一章中，你用 JavaScript 编写了一首歌！现在你已经习惯使用 Tone.js，你可以用它来创作自己的歌曲。另一个有趣的尝试是算法音乐，在这种方式下，你不是写出固定的歌曲，而是编写代码，每次运行时都会随机生成新的音乐。一种简单的尝试方法是列出一组听起来不错的和弦，然后随机选择某个和弦在某个特定的节拍上演奏（你可以使用 Tone.Loop 来实现这一点，就像我们在清单 12-12 中所做的那样）。
