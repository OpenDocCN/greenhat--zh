

## 第三章：11 面向对象的 PONG



![](img/opener.png)

在上一章中，我们构建了自己的*Pong*游戏版本。早些时候，在第六章中，你学习了 JavaScript 中的类和面向对象编程。你可能会想，为什么我们在*Pong*的实现中没有使用任何类。主要原因是我想让游戏代码尽可能简单，不加入任何不必要的概念，以便更容易理解实际游戏在做什么。然而，随着程序变得越来越大和复杂，给它们添加更多结构是很有帮助的，而一种常见的做法就是使用面向对象编程。

为了帮助你更好地理解如何以面向对象的风格设计软件，在本章中我们将演示一个面向对象版本的*Pong*。游戏的逻辑不会有任何变化，但代码的结构和组织方式将会不同。例如，处理球的代码将全部放在一个名为 Ball 的类中。我们将使用这个类来跟踪球的位置，并确定当球撞到墙壁或挡板时应该如何反弹。类似地，处理挡板的所有代码将放在一个名为 Paddle 的类中。通过让 Ball 和 Paddle 类继承自一个共享的父类，我们可以轻松地共享适用于球和挡板的公共代码。

本章将探索面向对象的*Pong*程序的一般结构，但我们不会深入到每一行代码的细节；你应该已经从上一章对它的工作原理有了相当好的理解。考虑到这一点，我们将不会逐步构建游戏，而是按照顺序逐节地讲解完整的代码。由于这个原因，代码在你输入完整之前不会正确运行或真正*执行*任何操作。但在我们进入代码之前，让我们首先更广泛地看一下如何设计面向对象的计算机程序。

### 面向对象设计

以面向对象的方式编写代码通过将代码组织成表示程序各个方面的类，给计算机程序增加了结构。这种结构使得其他程序员（甚至以后版本的你自己）更容易理解你的代码如何工作。面向对象设计技术的完整阐述超出了本书的范围，但在本节中，我们将探讨一些面向对象编程的核心关键原则。

面向对象设计的一个重要初步步骤是对你的领域或程序的世界进行建模。程序中有哪些不同的元素，它们需要做什么，以及它们如何相互关联和交互？在这个例子中，领域是游戏*Pong*，游戏中有几个可见的元素：球、球拍和得分。虽然有两个球拍，但它们的行为大致相同，因此我们可以创建一个 Paddle 类并实例化两个自定义对象。同时，球足够独特，值得有一个自己的类。我们还需要建模这些元素如何交互。例如，如何建模球与球拍的碰撞？这段代码必须放在某个地方。正如你将看到的，在我的设计中，我决定这段代码应该放在 Ball 类中。换句话说，球应该“知道”在与球拍和墙壁碰撞时如何反弹。

面向对象编程的另一个重要概念是*封装*。这意味着将类的内部细节隐藏起来，仅提供一个简单的接口供程序与类进行交互。封装这些细节使得我们可以在不影响程序其他部分的情况下，轻松地修改这些细节。例如，Ball 类不需要向程序的其他部分暴露它的速度或位置。如果我们决定改变速度的表示方式（例如，使用角度和速度代替 xSpeed 和 ySpeed），那么我们不需要修改程序的其他部分。

> 注意

*从技术上讲，xSpeed 和 ySpeed 将可以在 Ball 类外部访问，但我们不会访问它们，因此我们可以将其视为封装的细节。JavaScript 确实有一种声明属性为*私有*的方式，意味着这些属性不能在类外部访问，但在撰写本文时，这是一个新特性，并且并非所有浏览器都支持。*

面向对象编程的一个关键概念是*多态*，即如果一个方法期望接收某个类的对象，那么它也可以接收该类子类实例的对象。例如，在这一章中，你将看到一个 Entity 类，它有一个 draw 方法以及两个子类：Paddle 和 Ball。符合多态原则，任何使用 draw 方法的代码应该能够接收任何类型的 Entity 作为参数，而不需要关心我们传入的是 Ball 还是 Paddle。

最终，面向对象设计更多的是一种艺术，而非科学，而且有很多不同的实现方式。你应该将本章中的设计视为解决问题的一种可能方式，而不是“唯一正确的做法”。记住这一点后，让我们深入了解我们的面向对象*Pong*代码。

### 文件结构

面向对象版本的 *Pong* 的 HTML 与上一章完全相同，但 JavaScript 完全不同。如果你愿意，可以复制 *tennjs* 目录，删除 *script.js* 文件，并根据以下各节中的代码创建一个新的 *script.js* 文件。或者，你也可以直接删除现有 *tennjs* 目录中 *script.js* 文件的所有代码，并用新的面向对象代码替换它。不管哪种方式，更新后的 *script.js* 文件将由一系列类声明组成，之后是一些额外的代码来启动游戏。我们将依次查看每一部分代码。

### GameView 类

我们将声明的第一个类叫做 GameView。这个类负责玩家对游戏的视图，即游戏的显示方式。由于游戏使用画布进行渲染，GameView 类负责管理画布和绘图上下文。该类还负责将球和挡板等元素绘制到画布上，并显示“GAME OVER”文本。请参见清单 11-1 中的代码。

```
class GameView {
❶ constructor() {
    let canvas = document.querySelector("#canvas");
    this.ctx = canvas.getContext("2d");
 this.width = canvas.width;
    this.height = canvas.height;
    this.offsetTop = canvas.offsetTop;
  }

❷ draw(…entities) {
    // Fill the canvas with black
    this.ctx.fillStyle = "black";
    this.ctx.fillRect(0, 0, this.width, this.height);

  ❸ entities.forEach(entity => entity.draw(this.ctx));
    }

❹ drawScores(scores) {
    this.ctx.fillStyle = "white";
    this.ctx.font = "30px monospace";
    this.ctx.textAlign = "left";
    this.ctx.fillText(scores.leftScore.toString(), 50, 50);
    this.ctx.textAlign = "right";
    this.ctx.fillText(scores.rightScore.toString(), this.width - 50, 50);
  }

❺ drawGameOver() {
    this.ctx.fillStyle = "white";
    this.ctx.font = "30px monospace";
    this.ctx.textAlign = "center";
    this.ctx.fillText("GAME OVER", this.width / 2, this.height / 2);
  }
} 
```

清单 11-1：GameView 类

GameView 构造函数 ❶ 获取对画布及其绘制上下文的引用，并将其分别保存为名为 canvas 和 ctx 的属性。它还存储了一些绘图所需的值：画布的宽度和高度，以及画布相对于浏览器视口顶部的偏移量。

draw 方法 ❷ 使用了在第五章中介绍的剩余参数。通过这种方式，你可以传递多个参数给 draw，所有的参数将被收集到一个名为 entities 的数组中。每个参数都是表示游戏元素的对象：球和两个挡板。该方法首先绘制一个黑色矩形来清空画布，然后遍历元素数组，依次调用每个元素的 draw 方法 ❸，并将绘制上下文作为参数传递。只有当传递给 GameView.draw 的每个对象都有自己的 draw 方法时，这种方式才有效；我们将在下一节看到如何实现这一点。GameView 上的 draw 方法负责在每次游戏循环时将内容绘制到画布上，但它将实际绘制游戏元素的责任委托给表示这些元素的对象。实际上，游戏中的每个元素都“知道”如何绘制自己，而 GameView.draw 只是协调这些调用。

drawScores 方法 ❹ 接受一个包含两个分数的对象，并将它们绘制到画布上。这与上一章的得分绘制代码非常相似。主要区别在于，它不再依赖全局变量来获取画布的宽度，而是通过引用 this.width 来使用 GameView 类中的宽度属性。

drawGameOver 方法❺也与上一章中的相应函数大致相同，但它从 GameView 获取宽度和高度，而不是从全局变量获取。

### 游戏元素

接下来，我们将实现表示三种主要游戏元素的类：两个挡板和球。我们将从一个名为 Entity 的超类开始，它将作为 Paddle 和 Ball 子类的父类。Entity 类存在的目的是共享挡板和球的通用代码。这包括跟踪元素的大小和位置、计算元素的边界以进行碰撞检测，以及绘制元素。由于所有游戏元素都是矩形，因此无论是挡板还是球，这些代码都是相同的。这展示了面向对象编程的美妙之处：我们可以在超类中编写所有通用代码，然后让子类继承它。

示例 11-2 包含了 Entity 类的代码。

```
class Entity {
❶ constructor(x, y, width, height) {
    this.x = x;
    this.y = y;
    this.width = width;
    this.height = height;
    }

❷ boundingBox() {
    return {
      left: this.x,
      right: this.x + this.width,
      top: this.y,
      bottom: this.y + this.height
    };
  }

❸ draw(ctx) {
    ctx.fillStyle = "white";
    ctx.fillRect(this.x, this.y, this.width, this.height);
  }
} 
```

示例 11-2：Entity 类

Entity 构造函数❶接受表示实体左上角的 x 和 y 坐标，以及表示实体大小的宽度和高度。这些值会作为属性保存。

boundingBox 方法❷返回一个对象，包含实体的左、右、上、下边界。在上一章中，我们为每个实体在 checkCollision 函数中手动创建了这些对象。Entity 超类为我们提供了一种方便的方法，可以将这种常见的计算方式推广到球和挡板。

draw 方法❸接受一个绘图上下文，并使用构造函数中定义的属性绘制一个白色矩形。传递到 GameView 上的 draw 方法的对象都将是 Entity 的子类，而 Entity 类中的 draw 方法将为每个项调用。

#### Paddles 类

Paddle 类继承自 Entity 类。在示例 11-3 中声明。

```
class Paddle extends Entity {
❶ static WIDTH = 5;
  static HEIGHT = 20
  static OFFSET = 10;

❷ constructor(x, y) {
    super(x, y, Paddle.WIDTH, Paddle.HEIGHT);
  }
} 
```

示例 11-3：Paddle 类

这个类包含三个*静态属性*，这些属性是分配给类本身的，而不是类的单个实例。静态属性的值将在所有类实例之间共享。在这个例子中，尽管每个 Paddle 实例需要自己的 x 和 y 坐标，但每个 Paddle 对象应该具有相同的宽度、高度，以及与画布左右边缘的相同偏移。因此，我们将这些值定义为静态属性 WIDTH、HEIGHT 和 OFFSET，它们对应于上一章中的 PADDLE_WIDTH、PADDLE_HEIGHT 和 PADDLE_OFFSET 常量。

> 注意

*在类中没有直接的方法来定义静态常量，这就是为什么上一章中的常量现在技术上变成了变量。它们的名称都是大写字母，表示它们应该作为常量来处理。*

你可以使用 static 关键字声明静态属性。例如，我们使用 static WIDTH = 5 ❶ 声明了 WIDTH 静态属性。静态属性通过点表示法访问，就像实例的属性一样，不同的是你在点的左边使用类名，而不是 this 或实例的名称。例如，Paddle.WIDTH 访问 WIDTH 静态属性。

Paddle 构造函数 ❷ 只有两个参数：x 和 y。它使用 super 调用父类（Entity）的构造函数，并将 x 和 y 参数以及 Paddle.WIDTH 作为宽度参数，Paddle.HEIGHT 作为高度参数传递。

#### Ball 类

接下来是 Ball 类。它和 Paddle 类类似，都是继承自 Entity，但 Ball 有自己的逻辑来根据速度更新位置，并进行碰撞检测。清单 11-4 显示了该类代码的第一部分。

```
class Ball extends Entity {
❶ static SIZE = 5;

❷ constructor() {
    super(0, 0, Ball.SIZE, Ball.SIZE);
  ❸ this.init();
  }

❹ init() {
    this.x = 20;
    this.y = 30;
    this.xSpeed = 4;
    this.ySpeed = 2;
  }

❺ update() {
    this.x += this.xSpeed;
    this.y += this.ySpeed;
    }

❻ adjustAngle(distanceFromTop, distanceFromBottom) {
    if (distanceFromTop < 0) {
      // If ball hit near top of paddle, reduce ySpeed
      this.ySpeed -= 0.5;
    } else if (distanceFromBottom < 0) {
      // If ball hit near bottom of paddle, increase ySpeed
      this.ySpeed += 0.5;
    }
  } 
```

清单 11-4：Ball 类的开头

这个类有一个静态属性叫 SIZE，定义了球的宽度和高度 ❶。接下来是它的构造函数方法 ❷。和 Paddle 构造函数一样，Ball 构造函数首先做的事情是调用父类 Entity 的构造函数，这次传递 0 作为 x 和 y 参数，Ball.SIZE 作为宽度和高度参数。0 只是占位符；实际上，球每次都会从相同的位置开始（20，30）。这个定位由 Ball 类的 init 方法处理，它在构造函数中第一次被调用 ❸。init 方法本身用于设置球的初始位置和速度 ❹，就像上一章中的 initBall 函数一样。每当球需要重置为初始位置（得分后），这个方法会被调用。

接下来的方法 update 使用球的当前速度来更新其 x 和 y 位置 ❺。接着是 adjustAngle 方法 ❻，它等同于上一章中描述的 adjustAngle 函数。根据球与挡板碰撞的位置，它改变球的垂直速度（反弹角度）。

Ball 类的定义在清单 11-5 中继续，包含了碰撞检测的方法。

```
class Ball extends Entity {
--snip--
  checkPaddleCollision(paddle, xSpeedAfterBounce) {
  ❶ let ballBox = this.boundingBox();
    let paddleBox = paddle.boundingBox();

    // Check if the ball and paddle overlap vertically and horizontally
  ❷ let collisionOccurred = (
      ballBox.left< paddleBox.right &&
      ballBox.right  > paddleBox.left &&
      ballBox.top< paddleBox.bottom &&
      ballBox.bottom > paddleBox.top
    );

    if (collisionOccurred) {
      let distanceFromTop = ballBox.top - paddleBox.top;
      let distanceFromBottom = paddleBox.bottom - ballBox.bottom;
    ❸ this.adjustAngle(distanceFromTop, distanceFromBottom);
    ❹ this.xSpeed = xSpeedAfterBounce;
    }
  }

  checkWallCollision(width, height, scores) {
    let ballBox = this.boundingBox();

    // Hit left wall
  ❺ if (ballBox.left < 0) {
      scores.rightScore++;
      this.init();
    }
    // Hit right wall
  ❻ if (ballBox.right > width) {
      scores.leftScore++;
      this.init();
    }
    // Hit top or bottom walls
    if (ballBox.top < 0 || ballBox.bottom > height) {
    ❼ this.ySpeed = -this.ySpeed;
    }
  }
} 
```

清单 11-5：Ball 类的其余部分

checkPaddleCollision 方法与上一章的 checkCollision 和 checkPaddleCollision 函数有一些重叠。该方法接受两个参数：表示其中一个球拍的对象和 xSpeedAfterBounce。后者表示如果发生球拍反弹，我们应将 xSpeed 设置为的新值，并允许我们配置球是否应该总是从左球拍反弹到右侧，或从右球拍反弹到左侧。与上一章一样，我们要求球与左球拍碰撞时向右弹回，反之亦然，以避免球在“球拍内部”反弹的奇怪情况。

我们使用父类 Entity 中的 boundingBox 方法来获取球和球拍的边界框 ❶，并将它们分别存储为 ballBox 和 paddleBox。接下来，我们比较不同的边界框边缘，判断球和球拍之间是否发生了碰撞，并将结果保存在布尔变量 collisionOccurred 中 ❷。如果 collisionOccurred 为 true，我们调用 adjustAngle 方法，并根据边界框计算出的适当距离 ❸，然后将球的 xSpeed 设置为 xSpeedAfterBounce ❹。

最后，checkWallCollision 方法检查球与墙壁之间是否发生了碰撞。它接受游戏区域的宽度和高度以及表示得分的对象作为参数。如果球击中左墙 ❺或右墙 ❻，则相应的得分会增加，并通过调用 init 方法重置球。如果球击中上下墙，它会弹回 ❼。

### 得分和计算机类

得分类是一个简单的容器，用于跟踪当前的得分。计算机类包含用于跟踪球的逻辑。这两个类的代码在列表 11-6 中。

```
class Scores {
❶ constructor() {
    this.leftScore = 0;
    this.rightScore = 0;
   }
}

class Computer {
❷ static followBall(paddle, ball) {
    const MAX_SPEED = 2;
    let ballBox = ball.boundingBox();
    let paddleBox = paddle.boundingBox();

    if (ballBox.top < paddleBox.top) {
      paddle.y -= MAX_SPEED;
    } else if (ballBox.bottom > paddleBox.bottom) {
      paddle.y += MAX_SPEED;
    }
  }
} 
```

列表 11-6：得分和计算机类

得分构造函数 ❶将左右玩家的得分初始化为 0。我们本可以仅使用一个普通对象来表示得分，但使用类能让代码结构更加一致。

计算机类有一个名为 followBall 的方法，用于根据球的位置更新左侧球拍的位置。这是一个*静态方法*，意味着它不需要类的实例来调用。我们通过使用 static 关键字 ❷将其声明为静态方法，类似于声明静态属性。静态方法通过类名而不是实例名来调用，像这样：Computer.followBall(leftPaddle, ball)。

> 注意

*当某个类的实例需要存储特定的属性时，我们会创建该类的实例。计算机类没有任何属性，所以我们不需要为其创建实例。由于计算机类从未被实例化，它也不需要构造函数。*

我们本可以轻松地创建一个独立的函数来移动左侧挡板，但和 Scores 类一样，将代码保持在 Computer 类内有助于保持一致性。

### Game 类

我们最终来到了 Game 类，这是所有其他类（如果适用）被实例化并且被拼接在一起、协调工作的地方。请参见列表 11-7 查看代码的第一部分。

```
class Game {
  constructor() {
    this.gameView = new GameView();
    this.ball = new Ball();
  ❶ this.leftPaddle = new Paddle(Paddle.OFFSET, 10);
  ❷ this.rightPaddle = new Paddle(
      this.gameView.width - Paddle.OFFSET - Paddle.WIDTH,
      30
    );

  ❸ this.scores = new Scores();
    this.gameOver = false;

  ❹ document.addEventListener("mousemove", e => {
    this.rightPaddle.y = e.y - this.gameView.offsetTop;
    });
  }

  draw() {
  ❺ this.gameView.draw(
      this.ball,
      this.leftPaddle,
      this.rightPaddle
    );

  ❻ this.gameView.drawScores(this.scores);
  } 
```

列表 11-7：Game 类的第一部分

Game 构造函数首先实例化了 GameView、Ball 和 Paddle 类。leftPaddle 实例通过 Paddle.OFFSET 来设置其 x 坐标 ❶。rightPaddle 则通过 Paddle.OFFSET、Paddle.WIDTH 和 this.gameView.width 来确定其 x 坐标 ❷，这与我们在上一章计算右边挡板位置的方式类似。

在一个类内部实例化其他类是面向对象代码中的常见特性。这种技术被称为*组合*，因为我们在其他实例内部组合实例。

接下来，Game 构造函数实例化了 Scores ❸并将 gameOver 布尔值设置为 false。最后，它设置了一个 mousemove 事件监听器 ❹，当用户移动鼠标时更新右侧挡板的位置。在类构造函数中设置的事件监听器与我们在本书中看到的其他事件监听器一样：只要应用程序运行，它就会一直有效，并在检测到事件时触发其处理函数。

构造函数之后是 Game 类的 draw 方法，它负责绘制游戏的所有视觉元素。首先，该方法调用 this.gameView.draw ❺，传递了三个主要游戏元素：this.ball、this.leftPaddle 和 this.rightPaddle。这是对我们在列表 11-1 中看到的 GameView 类的 draw 方法的调用，它接收可变数量的对象作为参数，并对每个对象调用 draw 方法。最终的结果是，game.draw 调用 gameView.draw，进而调用 ball.draw、leftPaddle.draw 和 rightPaddle.draw。这个过程看起来有点绕，但你会发现面向对象代码中经常会有类似的情况，保持代码在逻辑上合适的位置有时需要绕过一些复杂的步骤。在这个例子中，game.draw 负责知道*哪些*对象需要绘制（因为 Game 类跟踪了所有的游戏元素）；gameView.draw 负责绘制上下文、清空画布，并调用各个元素的 draw 方法；而每个游戏元素的 draw 方法则负责知道如何绘制自身。

在绘制所有实体之后，draw 方法调用了 this.gameView.drawScores，并传递了 this.scores 对象 ❻。

Game 类在列表 11-8 中继续实现其剩余的方法。

```
class Game {
--snip--
  checkCollision() {
    this.ball.checkPaddleCollision(this.leftPaddle,
                                 ❶ Math.abs(this.ball.xSpeed));
    this.ball.checkPaddleCollision(this.rightPaddle,
                                 ❷ -Math.abs(this.ball.xSpeed));

  ❸ this.ball.checkWallCollision(
      this.gameView.width,
      this.gameView.height,
      this.scores
    );

  ❹ if (this.scores.leftScore > 9 || this.scores.rightScore > 9) {
      this.gameOver = true;
    }
  }

❺ update() {
    this.ball.update();
    Computer.followBall(this.leftPaddle, this.ball);
  }

❻ loop() {
    this.draw();
    this.update();
    this.checkCollision();

  ❼ if (this.gameOver) {
      this.draw();
      this.gameView.drawGameOver();
    } else {
      // Call this method again after a timeout
    ❽ setTimeout(() => this.loop(), 30);
    }
  }
} 
```

列表 11-8：Game 类的其余部分

Game 类的 checkCollision 方法协调所有的碰撞检测逻辑。首先，它调用球的 checkPaddleCollision 方法两次，以检查球与每个挡板之间的碰撞。回顾清单 11-5，这个方法接受两个参数：一个 Paddle 对象和一个新的、反弹后的 xSpeed 值。对于左侧挡板，我们知道我们希望球向右反弹，因此我们通过取当前 xSpeed 的 Math.abs 值来使新的 xSpeed 为正❶。对于右侧挡板，我们希望球向左反弹，因此我们通过取 Math.abs(xSpeed)的结果的负值来使新的 xSpeed 为负❷。

接下来，checkCollision 方法调用 ball.checkWallCollision 来处理墙壁碰撞❸。这个方法接受宽度和高度（因为 Ball 对象不知道游戏区域有多大）以及得分（如果撞到侧墙，就可以增加得分）。最后，方法检查是否有任何一个得分超过了阈值❹，如果超过，则将 this.gameOver 设置为 true。

Game 对象的 update 方法❺控制游戏循环每次重复时状态的变化。它调用球的 update 方法来移动球，然后通过 Computer.followBall 静态方法根据球的新位置告诉计算机移动左侧挡板。

Game 类的最后一个方法 loop 定义了游戏循环❻。我们按顺序调用 this.draw、this.update 和 this.checkCollision。然后，我们检查 this.gameOver 是否为 true。如果是❼，我们再次调用 draw 以渲染最终得分，并调用 gameView.drawGameOver 渲染“GAME OVER”文本。否则，我们使用 setTimeout 在 30 毫秒后再次调用 loop 方法❽，继续游戏。

### 开始游戏

我们需要做的最后一件事是通过实例化 Game 类并启动游戏循环来开始游戏，如清单 11-9 所示。

```
let game = new Game();
game.loop(); 
```

清单 11-9：开始游戏

我们必须在程序的顶层创建 Game 类的实例，而不是在任何类定义内部。所有其他所需的对象都是由 Game 类的构造函数实例化的，因此创建一个 Game 对象会自动创建所有其他对象。我们也可以让 Game 构造函数调用 loop 方法，以便在 Game 类实例化时就开始游戏。然而，将第一次调用 game.loop 放在程序的顶层可以更容易地看到游戏何时开始。

有了这个最终的清单，我们现在拥有了面向对象版本的游戏的所有代码！只要你按顺序输入所有代码，现在应该能正常运行，并且游戏玩法应与前一章的版本完全相同。

### 总结

在本章中，你创建了一个面向对象版本的*Pong*程序，并在此过程中学习了一些面向对象软件设计的策略。前一章中的游戏逻辑没有变化；只有代码的组织方式不同。根据你的偏好和面向对象代码的经验，你可能会发现这两种版本中的某一种更容易阅读和理解。

面向对象设计是一个复杂的领域，通常需要大量的实践才能将程序分解成各自独立且合理的对象。即使在这个简单的游戏中，你也可以用许多不同的方式将游戏的组件拆分成对象和方法。例如，你可能会认为 GameView 类是不必要的，Game 类本身就可以跟踪画布，从而避免复杂的绘制调用层层嵌套。最重要的是，以一种对你和其他程序员都易于理解和修改的方式来组织你的代码。
