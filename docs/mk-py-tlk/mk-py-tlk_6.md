# 第二章：B

章节末尾练习的建议答案

![](img/chapterart.png)

本附录提供了本书所有章节末尾练习的建议答案。如果在任何章节问题上遇到困难，您可以在这里查看答案并继续学习。虽然大多数章节都有章节末尾的练习，8、16 和 17 章没有，但它们有很多“动手做”的问题，帮助您进行练习。

## 第一章

1.  添加这一行代码：

    ```py
    print("Here is a third message!")
    ```

1.  以下是分别输出：

    ```py
    2
    9
    2
    2.3333333333333335
    1
    4
    20
    ```

1.  添加这一行代码：

    ```py
    print(55 * 234)
    ```

## 第二章

1.  以下是输出：

    ```py
    <class 'str'>
    <class 'str'>
    Kentucky Wildcats
    WildcatsKentucky 
    Wildcats @ Kentucky 
    WildcatsWildcatsWildcats
    ```

1.  输出如下：

    ```py
    <class 'float'>
    <class 'float'>
    3.46
    -2.4
    3.0
    ```

1.  以下是输出：

    ```py
    <class 'int'>
    57
    0.0
    ```

1.  输出如下：

    ```py
    <class 'bool'>
    8<7
    False
    <class 'str'>
    <class 'str'>
    ```

1.  这些是输出：

    ```py
    -23
    56
    -23.0
    8.0
    ```

1.  以下是输出：

    ```py
    1
    0.0
    False
    ```

1.  输出如下：

    ```py
    False
    True
    True
    True
    ```

1.  以下是答案：

1.  `global`：不行，因为它是 Python 的关键字

1.  `2print`：不行，因为变量不能以数字开头

1.  `print2`：是的

1.  `_squ`：是的

1.  `list`：不行，因为`list()`是 Python 内建函数

1.  这是输出：

    ```py
    this is A1
    this is A2
    ```

1.  输出如下：

    ```py
    this is A1
    this is A2
    this is C1
    this is C2
    ```

1.  这是输出：

    ```py
    this is A1
    this is A2
    this is B1
    this is B2
    this is C1
    this is C2
    ```

1.  答案如下：

    ```py
    0
    1
    2
    3
    4
    ```

    ```py
    10
    11
    12
    13
    14
    ```

    ```py
    10
    12
    14
    ```

1.  270

1.  请参见书籍资源中的新脚本*import_local_module1.py*。

1.  分数的范围和平均值分别是 29 和 82.625。您可以使用以下代码行来获取答案：

    ```py
    midterm = [95, 78, 77, 86, 90, 88, 81, 66]
    print("the range is", max(midterm)-min(midterm))
    print("the average is", sum(midterm)/len(midterm))
    ```

1.  输出如下：

    ```py
    rsity
    y
    University
    rsity of Kentucky
    ```

1.  如果`email = John.Smith@uky.edu`，则`email.find("y")`返回`13`。

1.  答案如下：

    ```py
    [2, 3, 5, 9]
    5
    3
    ```

1.  输出如下：

    ```py
    ["a", "hello", 2]
    [1, "a", "hello", 2, "hi"]
    ```

1.  `10`

1.  `(9,)`

1.  一种方式是使用`enumerate()`方法，如下所示：

    ```py
    lst = [1, "a", "hello", 2]
    newdict = {}
    for i, x in enumerate(lst):
        newdict[i] = x
    print(newdict)
    ```

    另一种方式如下：

    ```py
    lst = [1, "a", "hello", 2]
    newdict = {i:lst[i] for i in range(len(lst))}
    print(newdict)
    ```

## 第三章

1.  更改

    ```py
    if inp == "stop listening":
        print('Goodbye!')
    ```

    到

    ```py
    if inp == "`quit the script`":
        print('`Have a great day!`')
    ```

1.  将以下内容添加到脚本的末尾：

    ```py
    elif "open text" in inp: 
        inp = inp.replace('open text ','')
        myfile = f'{inp}.txt)'
        open_file(myfile)
        continue
    ```

1.  更改

    ```py
    import speech_recognition as sr 
    speech = sr.Recognizer()
    def voice_to_text():
        voice_input = "" 
        with sr.Microphone() as source:
            speech.adjust_for_ambient_noise(source)
            try:
                audio = speech.listen(source)
                voice_input = speech.recognize_google(audio)
     except sr.UnknownValueError:
                pass
            except sr.RequestError:
                pass        
            except sr.WaitTimeoutError:
                pass
        return voice_input 
    ```

    到

    ```py
    `from mysr import voice_to_text`
    ```

    确保将脚本*mysr.py*与*voice_open_file.py*放在同一文件夹中。

## 第四章

1.  请参见此处的*pyttsx3_adjust1.py*：

    ```py
    import pyttsx3
    engine = pyttsx3.init()
    voices = engine.getProperty('voices')
    engine.setProperty('voice', voices[0].id)
    engine.setProperty('rate', 160)
    engine.setProperty('volume', 0.8)
    engine.say("This is a test of my speech id, speed, and volume.")
    engine.runAndWait()
    ```

1.  请参见此处的*area_hs1.py*：

    ```py
    # Put mysr.py and mysay.py in the same folder as this script
    from mysr import voice_to_text
    from mysay import print_say
    # Ask the base length of the triangle
    print_say('What is the base length of the triangle?')
    # Convert the voice input to a variable inp1
    inp1 = voice_to_text()
    print_say(f'You just said {inp1}.')
    # Ask the height of the triangle
    print_say('What is the height of the triangle?')
    # Save the answer as inp2
    inp2 = voice_to_text()
    print_say(f'You just said {inp2}.')
    # Calculate the area
    area = float(inp1)*float(inp2)/2
    # Print and speak the result
    print_say(f'The area of the triangle is {area}.')
    ```

## 第五章

1.  更改

    ```py
    elif re2 == "too high":
        print_say("Is it 1?")
        while True:
            re3 = voice_to_text()
            print_say(f"You said {re3}.")
            if re3 in ("too high", "that is right", "too small"):
                break
        if re3 == "too small":
            print_say("It is 2!")
            sys.exit
        elif re3 == "that is right":
            print_say("Yay, lucky me!")
            sys.exit
    ```

    到

    ```py
    elif re2 == "too high":
        print_say("Is it `2`?")
        while True:
            re3 = voice_to_text()
            print_say(f"You said {re3}.")
            if re3 in ("too high", "that is right", "too small"):
                break
        if re3 == "`too high`":
            print_say("It is `1`!")
            sys.exit
        elif re3 == "that is right":
            print_say("Yay, lucky me!")
            sys.exit 
    ```

1.  更改

    ```py
    print(answer) 
    ```

    到

    ```py
    print(answer`[0:300]`)
    ```

1.  更改

    ```py
    from pygame import mixer 
    ```

    到

    ```py
    `import platform`
    ```

    并更改

    ```py
    mixer.init()
    mixer.music.load(f"./chat/{mysong}")
    mixer.music.play()
    ```

    到

    ```py
    `if platform.system() == "Windows":`
     `os.system(f"explorer ./chat/{mysong}")`
    `elif platform.system() == "Darwin":`
     `os.system(f"open ./chat/{mysong}")`
    `else:`
     `os.system(f"xdg-open ./chat/{mysong}")`
    ```

1.  更改

    ```py
    and "mp3" in file.name 
    ```

    到

    ```py
    and "`wav`" in file.name 
    ```

## 第六章

1.  请参见此处的*parse_local2.py*：

    ```py
    from bs4 import BeautifulSoup
    textfile = open("UKYexample.html", encoding='utf8')
    soup = BeautifulSoup(textfile, "html.parser")
    ptags = soup.findAll("p")
    atag = ptags[1].find("a")
    print(atag['class'])
    print(atag['href'])
    ```

1.  请参见此处的*scrape_live_web2.py*：

    ```py
    from bs4 import BeautifulSoup
    import requests
    url = 'http://libraries.uky.edu'
    page = requests.get(url)
    soup = BeautifulSoup(page.text, "html.parser")
    div = soup.find('div', class_="sf-middle")
    contact = div.find("div", class_="dashing-li-last")
    area = contact.find('span', class_="featured_area")
    print(area.text)
    atag = contact.find('span', class_="featured_email")
    print(atag.text)
    ```

1.  请参见此处的*voice_podcast.py*：

    ```py
    from io import BytesIO
    import requests
    import bs4
    from pygame import mixer
    # Import functions from the local package
    from mptpkg import voice_to_text, print_say
    def podcast():
        # Break a long url into multiple lines
        url = ('https://goop.com/the-goop-podcast/'
             'gwyneth-x-oprah-power-perception-soul-purpose/')
        # Convert the source code to a soup string
        response=requests.get(url)
        soup = bs4.BeautifulSoup(response.text, 'lxml')
        casts = soup.findAll\
        ('audio', {'class':'podcast-episode__audio-player'})
        casts = str(casts)
        start = casts.find("https")
        end = casts.find(".mp3")
        cast= casts[start:end+4]
        # Play the mp3 using the pygame module
        mymp3 = requests.get(cast)
        voice = BytesIO()
        voice.write(mymp3.content)
        voice.seek(0)
        mixer.init()
        mixer.music.load(voice)
        mixer.music.play()
    while True:
        print_say('Python is listening...')
        inp = voice_to_text().lower()
        print_say(f'you just said: {inp}')
        if inp == "stop listening":
            print_say('Goodbye!')
            break
        # If "podcast" in your voice command, play podcast
        elif "podcast" in inp: 
            podcast()
            # Python listens in the background
            while True:
                background = voice_to_text().lower()
                # Stops playing if you say "stop playing"
                if "stop playing" in background:
                    mixer.music.stop()
                    break
    ```

## 第七章

1.  以下是脚本：

    ```py
    import arrow
    from mptpkg import print_say

    dt = arrow.now().format('MMMM D, YYYY')
    tm = arrow.now().format('hh:mm:ss A')
    print_say(f'today is {dt}, and the time now is {tm}.')
    ```

1.  更改

    ```py
    elif "stop" in voice_input:
    ```

    到

    ```py
    elif "`quit the script`" in voice_input:
    ```

## 第九章

1.  变更如下：

    ```py
    import turtle as t

    t.Screen()
    t.setup(`500`,`400`,100,200)
    t.bgcolor('`blue`')
    t.title('`Modified Screen`')
    t.done() 
    t.bye()
    ```

1.  更改

    ```py
    t.forward(200)
    t.backward(300)
    ```

    到

    ```py
    `t.backward(100)`
    `t.forward(250):`
    ```

1.  变更如下：

    ```py
    import turtle as t

    t.Screen()
    t.setup(600,500,100,200)
    t.title('Python Turtle Graphics')
    t.hideturtle()
    t.up()
    `t.goto(100,100)`
    `t.dot(60,'lightgreen')`
    `t.goto(-100,-100)`
    `t.dot(60,'lightgreen')`
    t.done()
    try:
        t.bye()
    except Terminator:
        pass
    ```

1.  更改

    ```py
    t.pencolor('blue')
    t.pensize(5)
    ```

    到

    ```py
    t.pencolor('`red`')
    t.pensize(`3`)
    ```

1.  代码如下：

    ```py
    import turtle as t
    t.Screen()
    t.setup(600,500,100,200)
    t.bgcolor('green')
    t.title('Python Turtle Graphics')
    t.hideturtle()
    t.tracer(False)
    t.pensize(6)
    t.goto(200,0)
    t.goto(200,100)
    t.goto(0,100)
    t.goto(0,0)
    t.update()
    t.done()
    try:
        t.bye()
    except Terminator:
        pass
    ```

## 第十章

1.  更改

    ```py
    t.goto(center)
    t.write(cell,font = ('Arial',20,'normal'))
    ```

    到

    ```py
    t.goto(`(`center`[0]-80, center[1]-80)`)
    t.write(cell,font = ('Arial',`15`,'normal'))
    ```

1.  更改

    ```py
    print(f'(x, y) is ({x}, {y})')
    ```

    到

    ```py
    print(f'(x, y) is ({x}, {y})')
    `print('x+y is ', x+y)`
    ```

1.  更改

    ```py
    print('row number is ', row)
    ```

    到

    ```py
    `print(f'you clicked on the point ({x}, {y})')`
    print('row number is ', row)
    ```

1.  更改

    ```py
    # The blue player moves first
    turn = "blue"
    ```

    到

    ```py
    # The `white` player moves first
    turn = `"white"`
    ```

1.  删除脚本中的以下内容：

    ```py
    if '1' in occupied[turn] and '5' in occupied[turn] and '9' in occupied[turn]:
        win = True
    if '3' in occupied[turn] and '5' in occupied[turn] and '7' in occupied[turn]:
        win = True
    ```

## 第十一章

1.  更改

    ```py
    done()
    ```

    到

    ```py
    `rownum = 1`
    `for y in range(-250, 300, 100):`
     `goto(325,y)`
     `write(rownum,font=('Arial',20,'normal'))`
     `rownum += 1`
    done()
    ```

1.  更改

    ```py
    sleep(0.05)
    ```

    到

    ```py
    sleep(`0.025`)
    ```

1.  删除`vertical4()`函数的定义，并删除脚本中的以下内容：

    ```py
    if vertical4(x, y, turn) == True:
        win = True
    ```

1.  将以下内容添加到字典`to_replace`：

    ```py
    'column ':'',
    ```

1.  请参见[`nostarch.com/make-python-talk/`](https://nostarch.com/make-python-talk/)中的*conn_hs_2player.py*。

## 第十二章

1.  更改

    ```py
    coins[i].goto(-100 + 50 * i, 0)
    ```

    到

    ```py
    coins[i].goto(-100 + 50 * i, `-10`)
    ```

1.  更改

    ```py
    coins[-(i+1)].hideturtle()
    ```

    到

    ```py
    coins[`i`].hideturtle()
    ```

1.  ```py
    ['H', 'i', ' ', 'P', 'y', 't', 'h', 'o', 'n'] 
    ```

## 第十三章

1.  更改

    ```py
    # The red player moves first
    turn = "red"
    ```

    到

    ```py
    # The `yellow` player moves first
    turn = "`yellow`"
    ```

    然后，删除

    ```py
    # Computer moves first
    computer_move()
    ```

    同时，删除

    ```py
    # Take column 4 in the first move
    if len(occupied[3]) == 0:
        return 4
    ```

    完整的脚本在[`nostarch.com/make-python-talk/`](https://nostarch.com/make-python-talk/)中的*conn_think1_second.py*里。

1.  请参见书籍资源网站中的*ttt_think1.py*。

1.  请参见书籍资源网站中的*ttt_think2.py*。

1.  值如下：

    ```py
    cnt = {2:7, 1:5, 4:6}
    ```

    请注意，由于`4`和`5`这两个值各自出现一次，因此它们在字典`cnt`中只会显示一个，因为字典不能有相同键的两个元素：`maxcnt = 4`，并且`cnt[maxcnt] = 6`。

1.  请参见书籍资源网站中的*ttt_think.py*。

1.  请参阅书籍资源网站上的*ttt_simulation.py*和*ttt_ml.py*。

1.  请参阅书籍资源网站上的*outcome_ttt_think.py*和*outcome_ttt_ml.py*。

1.  黄色玩家赢得了第 10 局比赛。四个棋盘上的棋子水平连接（在第 2、3、4 和 5 列）。

## 第十四章

1.  更改

    ```py
    start_date = "2020-09-01"
    end_date = "2021-02-28"
    ```

    为

    ```py
    start_date = "`2021-03-01`"
    end_date = "2021-`06-01`"
    ```

    并更改

    ```py
    plt.plot(stock['Date'], stock['Adj Close'], c = 'blue')
    ```

    为

    ```py
    plt.plot(stock['Date'], stock['Adj Close'], c = '`red`')
    ```

1.  更改

    ```py
    formatter = mdates.DateFormatter('%m/%d/%Y')
    ```

    为

    ```py
    formatter = mdates.DateFormatter('%m`-`%d`-`%Y') 
    ```

    并更改

    ```py
    plt.setp(fig.get_xticklabels(), rotation = 10)
    ```

    为

    ```py
    plt.setp(fig.get_xticklabels(), rotation = `15`)
    ```

## 第十五章

1.  更改

    ```py
    usd = response_json['bpi']['USD']
    # Get the price
    price = usd['rate_float']
    print(f"The Bitcoin price is {price} dollars.")
    ```

    为

    ```py
    `gbp` = response_json['bpi']['`GBP`']
    # Get the price
    price = `gbp`['`rate`']
    print(f"The Bitcoin price is {price} `pounds`.")
    ```

1.  更改

    ```py
    root.geometry("800x200")
    # Create a label inside the root window
    label=tk.Label(text="this is a label", fg="Red", font=("Helvetica", 80))
    ```

    为

    ```py
    root.geometry("`850x160`")
    # Create a label inside the root window
    label=tk.Label(text="`here is your label`", fg="Red", font=("Helvetica", 80))
    ```

1.  更改

    ```py
    root.after(1000, bitcoin_watch)
    ```

    为

    ```py
    root.after(`800`, bitcoin_watch)
    ```

1.  更改

    ```py
    maxprice = oldprice * 1.05
    minprice = oldprice * 0.95
    ```

    为

    ```py
    maxprice = oldprice * `1.03`
    minprice = oldprice * `0.97`
    ```
