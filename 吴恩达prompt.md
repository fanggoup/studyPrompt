# 吴恩达prompt

## 1.两种类型的LLM

1. base LLM

   基于文本训练数据来预测做“文字接龙”

2. instruction tuned LLM

   接受了遵循指示的培训

   训练过程：首先使用已经在大量文本数据上训练过的基本LLM，然后使用输入和输出的指令来进行微调

   (RLHF)

## 2.指南

### 两个关键原则

#### 1.编写明确和具体的指令

目的：减少不正确的回答

`清楚`的提示与简短提示是不一样



方法:

- ##### 使用分隔符清楚地指示输入的不同部分：使用分隔符可以避免提示词的冲突

  ```python
  text = f"""
  You should express what you want a model to do by \
  provideing instructions that are clear and \
  speacific as you can possibly make them. \
  This will guide the model towars the desired output, \ 
  and reduce the chances of receiving irrelevant \
  or incorrect responses. Don't confuse wiriting a \
  clear prompt with writing a short prompt. \
  In many cases,linger prompts provide more clearity \
  and context for the model, which can lead to \
  more detailed and relevant outputs.
  """
  prompt = f"""
  summarize the text delimited by triple backticks \
  into a single sentence.
  ​```{text}```
  """
  response = get_completion(prompt)
  print(response)
  ```

  提示冲突：如果允许用户向提示中添加一些输入，可能会给出与我们想要的任务不符的指令，导致模型遵循用户的指令而不是我们想要的指令

- ##### 要求结构化输出：为了使解析模型输出更容易，可以请求使用HTML或JSON等结构化输出

  ```python
  promt = f"""
  Generate a list of three made-up book titles along \
  with their autors and genres.
  Provide the in JOSN format with the following keys:
  bookid,title,author,genre.
  """
  response = get_completion(prompt)
  print(response)
  ```

  

- ##### 要求模型检查是否满足条件：如果任务存在假设未必满足，我们可以告诉模型首先检查这些假设，如果不满足，则指示停止尝试完全完成任务

  ```python
  text_1 = f"""
  Making a cup of tea is easy!First ,you need to get some \
  water boiling.While that's happening, \
  grad a cup and put a tea bag in it.Once the water is \
  hot enough,just pour it over the tea bag.\
  Let it sit for a bit so the tea can steep.After a \
  few minutees,take out the tea bag.If you \
  like ,you can add some sugar or milk to taste. \ 
  And that's it! You've got yourself a delicious \
  cup of tea to enjoy.
  """
  prompt = f"""
  You will be provided with text delimited by triple quotes.
  If it contains a sequence of instrucions, \
  re-write those instructions in the following format:
  
  Step 1 - ...
  Step 2 - …
  …
  Step N - …
  …
  If the text does not contain a sequence of instructions, \
  then simply write \"No steps provided.\"
  
  \"\"\"{text_1}\"\"\"
  """
  response = get_completion(prompt)
  print("Completion for Text 1:")
  print(response)
  ```

  

- ##### 少量训练提示：要求模型执行任务之前，提供成功执行任务的示例

  ```python
  prompt = f"""
  Your task is to answer in a consistent style.
  
  <child>:Teach me about patience.
  
  <grandparent>: The river that carves the deepest \
  valley flows from a modest spring;the \
  grandest symphony originates from a single notes; \
  the most intricate tapestry begins with a solitary thread.
  
  <child>: Teach me about resilience.
  """
  response = get_completion(prompt)
  print(response)
  ```

  

  

  

#### 2.给模型足够的时间思考

- ##### 指定完成任务所需的步骤

  ```python
  text = f"""
  In a charming village,siblings Jack and Jill set out on \
  a quest to fetch water from a hilltop \
  well. As they climbed, singing joufully, misfortune \
  struck-Jack tripped on a stone and tumbled \
  down the hill, with Jill following suit. \
  Though slightly battered, the pair returned home to \
  comforting embraces. Despite the mishap, \
  their adventurous spirits remained undimmed, and they \
  continued exploring with delight.
  """
  
  #example 1
  prompt_1 = f"""
  Perform the following actions:
  1 - Summarize the following text delimited by triple \
  backticks with 1 sentence.
  2 - Translate the summary into French.
  3 - List each name in the French summary.
  4 - Output a json object that contains the folloeing \
  keys: french_summary, num_names.
  
  Separate yout answers with line breaks.
  
  Text:
  ​```{text}```
  """
  response = get_completion(prompt_1)
  print("Completion for prompt 1:")
  print(response)
  ```

```python
#example 2,asking for output in a specified format
prompt_2 = f"""
Perform the following actions:
1 - Summarize the following text delimited by triple \
quotes with 1 sentence.
2 - Translate the summary into French.
3 - List each name in the French summary.
4 - Output a json object that contains the folloeing \
keys: french_summary, num_names.

Use the following format:
Text:<text to summarize>
Summary:<summary>
Translation:<summary translation>
Names:<list of names in Italian summary>
Output JSON:<json with summary and num_names>

Text: <{text}>
"""
response = get_completion(prompt_2)
print("Completion for prompt 2:")
print(response)
```

- ##### 指示模型在匆忙做出结论之前思考解决方案

  ```python
 prompt = f"""
  Determine if the student's solution is correct or not.
  
  Question:
  I'm building a solar power installation and I need \
  help working out the financials.
  - Land costs $100 / square foot
  - I can buy solar panels for $250 / square foot
  - I negotiated a contract for maintenance that will cost \
  me a flat $100k per year, and an additional $10 / square \
  foot
  What is the total cost for the first year of operations
  as a function of the number of square feet .
  
  Student's Solution:
  Let x be the size of the installation in square feet.
  Costs:
  1. Land cost: 100x
  2. Solar panel cost: 250x 
  3. Maintenance cost: 100,000 + 100x
  Total cost: 100x + 250x + 100,000 + 100x = 450x + 100,000
  """
  response = get_ completion (prompt)
  print(response)
  
  ```

  ```python
  prompt = f"""
  Your task is to determine if the student's solution \
  is correct or not.
  To solve the problem do the following:
  - First, work out your own solution to the problem.
  - Then compare your solution to the student's solution \
  and evaluaterif the student's solution is correct or not.
  Don't decidetif the student's solution is correct until
  you have done the problem yourself .
  
  Use the following format:
  Question :
  ​```
  question here
  ​```
  Student's solution:
  ​```
  student's solution here
  ​```
  Actual solution:
  ​```
  steps to work out the solution and your solution here
  ​```
  Is the student's solution the same as actual solution \
  just calculated:
  ​```
  yes or no
  ​```
  Student grade:
  ​```
  correct or incorrect
  ​```
  
  Question:
  ​```
  I'm building a solar power installation and I need \
  help working out the financials.
  - Land costs $100 / square foot
  - I can buy solar panels for $250 / square foot
  - I negotiated a contract for maintenance that will cost \
  me a flat $100k per year, and an additional $10 / square \
  foot
  What is the total cost for the first year of operations
  as a function of the number of square feet .
  
  Student's Solution:
  Let x be the size of the installation in square feet.
  Costs:
  1. Land cost: 100x
  2. Solar panel cost: 250x 
  3. Maintenance cost: 100,000 + 100x
  Total cost: 100x + 250x + 100,000 + 100x = 450x + 100,000
  """
  response = get_ completion (prompt)
  print(response)
  ```



### 模型限制

问题：Hallucination(幻觉)

尝试回答不会的问题并编造看似合理实际上不正确的内容



解决：减少幻觉

要求模型首先从文本中找到任何相关的引用，然后要求它使用这些引用来回答问题



## 3.了解迭代提示开发过程

原因：第一次就得到最合适的prompt可能性比较小，所以最重要的是开发适合自己的应用程序的prompt的过程

提示开发是一个迭代过程：

1. 尝试一些东西
2. 看看结果没有实现我们想要的目标
3. 考虑如何说清楚指令，或者考虑如何给它更多的思考空间，以便接近我们所需的结果
4. 使用一批示例优化提示



测试：

```python
fact = '''
接下来你的任务是根据设计说明书帮助营销部门为零售网站或产品创建描述，编写产品说明书。

设计说明书：
儿童书桌技术说明书

一、作用

儿童书桌是为儿童提供一个舒适的学习环境，帮助他们更好地学习和成长。它可以让孩子们在一个安静、整洁的空间里专心学习，提高学习效率和兴趣。

二、结构

儿童书桌主要由桌面、桌腿、抽屉和书架等部分组成。桌面一般采用环保材料，如MDF板、实木板等，表面覆盖防水、防刮、防污的材料，易于清洁。桌腿一般采用钢管或实木材料，具有稳定性和承重能力。抽屉和书架可以根据需要选择，方便储存书籍、文具等物品。

三、尺寸

儿童书桌的尺寸应根据儿童的身高和年龄来确定。一般来说，桌面高度应该与儿童肘部高度相当，桌面宽度应该能够容纳书籍、文具等物品。桌面长度应该根据儿童的身高和学习需求来确定，一般为80-120厘米。

四、型号

儿童书桌的型号可以根据不同的需求来选择。一般来说，有单人书桌、双人书桌、可升降书桌等多种型号可供选择。在选择时，应根据儿童的年龄、身高、学习需求和空间大小等因素来确定。

以上是儿童书桌的技术说明书，希望能够帮助您选择适合的书桌，为孩子提供一个良好的学习环境。
'''
response = get_completion(fact)
print(response)


结果：
产品说明书：

儿童书桌是一款专为儿童设计的学习桌，旨在为孩子们提供一个舒适、整洁的学习环境，帮助他们更好地学习和成长。

该书桌采用环保材料制作，桌面采用MDF板或实木板，表面覆盖防水、防刮、防污的材料，易于清洁。桌腿采用钢管或实木材料，具有稳定性和承重能力。抽屉和书架可以根据需要选择，方便储存书籍、文具等物品。

儿童书桌的尺寸应根据儿童的身高和年龄来确定。桌面高度应该与儿童肘部高度相当，桌面宽度应该能够容纳书籍、文具等物品。桌面长度应该根据儿童的身高和学习需求来确定，一般为80-120厘米。

该书桌有多种型号可供选择，包括单人书桌、双人书桌、可升降书桌等。在选择时，应根据儿童的年龄、身高、学习需求和空间大小等因素来确定。

我们相信，儿童书桌将为孩子们提供一个良好的学习环境，帮助他们更好地学习和成长。
```

```python
fact = '''
接下来你的任务是根据设计说明书帮助营销部门为零售网站或产品创建描述，编写产品说明书。

这个描述是为家具零售商准备的，所以它应该是技术性的，关注椅子的材料、产品和结构。

设计说明书：
儿童书桌技术说明书

一、作用

儿童书桌是为儿童提供一个舒适的学习环境，帮助他们更好地学习和成长。它可以让孩子们在一个安静、整洁的空间里专心学习，提高学习效率和兴趣。

二、结构

儿童书桌主要由桌面、桌腿、抽屉和书架等部分组成。桌面一般采用环保材料，如MDF板、实木板等，表面覆盖防水、防刮、防污的材料，易于清洁。桌腿一般采用钢管或实木材料，具有稳定性和承重能力。抽屉和书架可以根据需要选择，方便储存书籍、文具等物品。

三、尺寸

儿童书桌的尺寸应根据儿童的身高和年龄来确定。一般来说，桌面高度应该与儿童肘部高度相当，桌面宽度应该能够容纳书籍、文具等物品。桌面长度应该根据儿童的身高和学习需求来确定，一般为80-120厘米。

四、型号

儿童书桌的型号可以根据不同的需求来选择。一般来说，有单人书桌、双人书桌、可升降书桌等多种型号可供选择。在选择时，应根据儿童的年龄、身高、学习需求和空间大小等因素来确定。

以上是儿童书桌的技术说明书，希望能够帮助您选择适合的书桌，为孩子提供一个良好的学习环境。
'''
response = get_completion(fact)
print(response)


结果：
产品说明书：

产品名称：儿童书桌

产品特点：

1.环保材料：采用MDF板、实木板等环保材料，表面覆盖防水、防刮、防污的材料，易于清洁。

2.稳定性：桌腿采用钢管或实木材料，具有稳定性和承重能力。

3.储物功能：配有抽屉和书架，方便储存书籍、文具等物品。

4.尺寸合适：桌面高度与儿童肘部高度相当，桌面宽度能够容纳书籍、文具等物品，桌面长度根据儿童的身高和学习需求来确定。

5.多种型号：有单人书桌、双人书桌、可升降书桌等多种型号可供选择。

产品用途：

儿童书桌是为儿童提供一个舒适的学习环境，帮助他们更好地学习和成长。它可以让孩子们在一个安静、整洁的空间里专心学习，提高学习效率和兴趣。

使用方法：

1.根据儿童的身高和年龄选择合适的型号和尺寸。

2.将桌腿安装到桌面上，确保稳定性。

3.根据需要选择抽屉和书架，方便储存书籍、文具等物品。

注意事项：

1.使用时应注意安全，避免儿童在使用过程中受伤。

2.定期清洁桌面和储物部分，保持整洁。

3.避免长时间暴露在阳光下或潮湿的环境中，以免影响使用寿命。

4.如有任何问题，请及时联系售后服务。

我们致力于为您提供优质的产品和服务，希望儿童书桌能够为您的孩子提供一个良好的学习环境。
```

```
将所有内容格式化为可以在网站中使用的 HTML。 将说明放在 <div>元素中。

<div>
<h2>儿童书桌技术说明书</h2>
<p>儿童书桌是为儿童提供一个舒适的学习环境，帮助他们更好地学习和成长。它可以让孩子们在一个安静、整洁的空间里专心学习，提高学习效率和兴趣。</p>
<h3>结构</h3>
<p>儿童书桌主要由桌面、桌腿、抽屉和书架等部分组成。桌面一般采用环保材料，如MDF板、实木板等，表面覆盖防水、防刮、防污的材料，易于清洁。桌腿一般采用钢管或实木材料，具有稳定性和承重能力。抽屉和书架可以根据需要选择，方便储存书籍、文具等物品。</p>
<h3>尺寸</h3>
<p>儿童书桌的尺寸应根据儿童的身高和年龄来确定。一般来说，桌面高度应该与儿童肘部高度相当，桌面宽度应该能够容纳书籍、文具等物品。桌面长度应该根据儿童的身高和学习需求来确定，一般为80-120厘米。</p>
<h3>型号</h3>
<p>儿童书桌的型号可以根据不同的需求来选择。一般来说，有单人书桌、双人书桌、可升降书桌等多种型号可供选择。在选择时，应根据儿童的年龄、身高、学习需求和空间大小等因素来确定。</p>
<p>以上是儿童书桌的技术说明书，希望能够帮助您选择适合的书桌，为孩子提供一个良好的学习环境。</p>
</div>
```

```python
from IPython.display import display,HTML
display(HTML(response))
```

![image-20230524110615389](C:\Users\发、\AppData\Roaming\Typora\typora-user-images\image-20230524110615389.png)

## 4.摘要

原因：没有足够的时间去阅读我们想要阅读的所有内容

```python
prompt = """
你的任务是从电子商务网站的产品评论中生成一个简短的总结，最多以30个字总结以下评论。

评论：华为matepad 11 2023款推荐购买柔光版，屏幕提升真的很爽，类纸的质感磨砂的摸起来很舒服，比玻璃面更舒适，保护眼睛。而且升级成骁龙870性价比更高
相比ipad 11来说屏幕小了一点点，不过有华为生态的话用着还是很爽的，多屏协同很好用
"""
response = get_completion(prompt)
print(response)

结果：
华为matepad 11 2023款柔光版，屏幕升级舒适保护眼睛，骁龙870性价比更高，华为生态多屏协同很爽。
```

```python
prompt = """
你的任务是从电子商务网站的产品评论中生成一个简短的总结，此总结是反馈给交通部门，应专注于提到运输和产品交付方面。最多以30个字总结以下评论。

评论：
外观材质：很薄，质感好。
屏幕效果：纠结要不要上Pro。Oled屏，看看这个其实也不错。
运行速度：新机目前很快很流畅，120的刷新率，可以的。
其他特色：昨天晚上10点多下的单，第二天睡醒九点就送到手里了。儿子学校学习要用平板，想想还是支持国货吧。拿到后的喜悦之情溢于言表?
"""
response = get_completion(prompt)
print(response)

结果：
快速交付，薄质感好，支持国货。
```

```python
prompt = """
你的任务是从电子商务网站的产品评论中提取关于运输和产品交付方面信息，信息是反馈给交通部门。

评论：京东物流送货上门次日即达。商品包装很好没有破损。平板外观颜色很好，屏幕看着舒适，重量还行，性价比还不错，华为品牌值得购买……
"""
response = get_completion(prompt)
print(response)

结果：
从这条评论中，我们可以得到以下关于运输和产品交付方面的信息：

- 京东物流送货上门次日即达，说明物流速度较快。
- 商品包装很好没有破损，说明物流过程中包装保护措施得当。
- 没有提到产品交付时是否需要签收等信息。
- 没有提到是否有物流跟踪信息可供查询。
- 以上信息都是正面的，没有提到任何负面的运输和产品交付方面的问题。
```



多条评论：采用循环调用

## 5.推理

提取标签、提取名称，理解文本情感（正面或负面）等方面的任务

```python
prompt = """
以下评论的情感是什么？

用“积极”或“消极”给出你的答案

评论：轻松了，就剩我一个人了。
"""
response = get_completion(prompt)
print(response)

结果：
消极
情感：孤独/失落。
```



```python
prompt = """
从评论中提取以下部分的信息：
- 手机品牌
- 评论者使用的手机版本

使用JSON格式进行回答，“品牌”和“版本”作为keys。

如果以上信息不存在，使用“不知道”作为value

你的回答应尽量简洁

评论：今天终于抢到了 20pro 版本， 对于一个用了七年 flm 的来说 这一切还不算晚，\
还记得一六年刚拥有的 nteo 5，用了一个星期就喜欢上了魅族，感觉这个系统很好，对我来说很方便，而且那时候的白色面板也是很好看。\
中间有用过 16 17 系列到如今的 20。魅族手机一直给我的感觉都是独特的，像身边的朋友很多都用苹果，\
而就我一个坚持着用了魅族，他们都有都劝说让我试下苹果。我都没有动摇过，一直支持着魅族
"""
response = get_completion(prompt)
print(response)

结果：
{
  "品牌": "魅族",
  "版本": "20pro"
}
```



```python
story = """
维基百科中对卫星互联网的定义：卫星上网是指由通讯卫星提供的网络存取服务。卫星上网通常需要三大部件。一颗通常位于地球静止轨道的卫星。
一个地面站，通常作为网关。还有是天线。卫星上网通信颗以分为双通道通信和单通道仅接收通信。

《宽带卫星通信互联网与卫星互联网》中对卫星互联网的定义：
以 VSAT 系统为基础、具有广播功能、以 IP为网络服务平台、以互联网应用为服务对象，能够成为互联网的一个组成部分，并能够独立运行的网络系统为卫星互联网，它又称为广播互联网。

《卫星互联网综述》中对卫星互联网的定义：
卫星互联网是基于卫星通信系统，以 IP 为网络服务平台，以互联网应用为服务对象，能够成为互联网的一个组成部分，并能够独立运行的网络系统。

当下新兴的卫星互联网星座，指新近发展的、能提供数据服务、实现互联网传输功能的巨型通信卫星星座。
新兴卫星互联网星座具有以下特点：从星座构成看，是由成百上千颗卫星组成的巨型星座；
从星座构成看，是由运行在非对地静止轨道（NGSO，包括低轨道和中轨道）数量众多的卫星构成；
从提供的服务看，主要是宽带的互联网接入服务；从发展卫星互联网星座的企业看，主要是非传统航天领域的互联网企业；
从项目发展的起始时间看，主要是在2014年底至2015年初开始的。

NGSO卫星系统具有覆盖范围广、通信容量大、传输延迟低的优点，正在改变当今的卫星通信。
根据上述定义，我们可以得出卫星互联网是面向互联网的蓬勃发展，
针对地面网络的不足（如覆盖受限、难以支持高速移动用户应用、广播类业务占用网络资源较多、易受自然灾害影响等），
利用卫星通信覆盖广、容量大、不受地域影响、具备信息广播优势等特点，作为地面通信的补充手段实现用户接入互联网，
可有效解决边远散、海上、空中等用户的互联网服务问题。
"""

prompt = """
确定以下文本中正在讨论的五个主题。

每个主题使用一个或两个词概括。

使用列表展示你的回答

文本：
"""+story
response = get_completion(prompt)
print(response)

结果：
1. 卫星上网
2. 卫星互联网定义
3. 新兴卫星互联网星座
4. NGSO卫星系统
5. 卫星互联网的优势和应用场景
```



零样本学习算法（无监督学习）

```python
topic_list = [
    "卫星系统","北斗","卫星互联网发展现状"
]
prompt = """
确定以下主题列表中的每个项目是否是文本中的主题。

在每个主题下给出你1或0的答案。

主题列表:
""" + ",".join(topic_list) + """
文本:
"""+story
response = get_completion(prompt)
print(response)

结果：
卫星系统：1
北斗：0
卫星互联网发展现状：1
```



```python
prompt = """
确定以下主题列表中的每个项目是否是文本中的主题。

对每个主题的答案以1或0表示。

最终的答案将以JSON的形式输出。

主题列表:
""" + ",".join(topic_list) + """
文本:
"""+story
response = get_completion(prompt)
print(response)
```



## 6.转换

将其输入转换为不同的格式（一段文本从一种语言翻译成另一种语言，帮助拼写或语法纠正，或者转换格式，比如输入html，输出JSON）

1.翻译

```python
prompt = """
将下面的文本翻译为日文：\
​```初次见面，你好```
"""
response = get_completion(prompt)
print(response)

结果
はじめまして、こんにちは。
```

2.判断语言

```python
prompt = """
下面这段文本是什么语言：\
​```はじめまして、こんにちは。```
"""
response = get_completion(prompt)
print(response)

结果：
这是日语
```

3.正式与非正式

```python
prompt = """
将下面文本翻译成中文的正式和非正式形式：\
`はじめまして、こんにちは。`
"""
response = get_completion(prompt)
print(response)

正式：初次见面，您好。
非正式：你好，初次见面。
```



多种语言翻译成同一种:循环调用



4.语气转换

```python
prompt = """
将下面文本翻译成商业信函：
`妈咪，你好，可以问一下宝贝多少米吗？`
"""
response = get_completion(prompt)
print(response)

尊敬的客户，

您好！感谢您对我们产品的关注。请问您能告诉我们宝贝的价格吗？

谢谢！

此致

敬礼

XXX公司
```



5.拼写检查和语法检查

```python
prompt = """
检查下面语句的语法和错误，并对其进行修正：
`is it sonw today？`
"""
response = get_completion(prompt)
print(response)

"Is it snowing today?"
```



## 7.扩展

根据一些信息生成个性化电子邮件



```python
review= """
千万别买了，色差很多，布料也很差，不要看好评多被骗了，根本广告的照片不是一件衣服，
买了不穿也浪费哈，自己可以对比一下这两件衣服的颜色，商家客服说是正常的色差。
"""
sentiment = "消极"

prompt = """
你是一位客户服务AI助手，你的任务是给你的客户发送一封电子邮件回复，给定客户电子邮件。
感谢客户的评论，如果情感是积极或中性，感谢他们的评论；
如果情感是消极的，道歉并建议他们联系客户服务。
确保使用评论中的具体细节。
用简明和专业的语气写信。
并签名为`AI客户代理`。

客户评论：
"""+review
"评论情感:"+sentiment
# +story
response = get_completion(prompt)
print(response)

结果：
尊敬的客户，

感谢您的评论。我们非常抱歉您对我们的产品感到失望。我们理解您对产品质量的期望，我们深感遗憾未能满足您的期望。

我们会尽快处理您的订单并退还您的款项。我们会认真考虑您的反馈，并采取措施改进我们的产品和服务。

如果您有任何其他问题或需要进一步的帮助，请随时联系我们的客户服务团队。我们将竭诚为您服务。

再次感谢您的反馈和支持。

AI客户代理
```



使用语言模型的一个参数，称为温度。它将允许我们改变模型响应的多样性。可以将温度视为模型的探索成都或随机性

在构建需要可预测回复的应用程序时，建议温度为0；

如果想要以更有创意的方式使用模型，获得更广泛的不同输出，可能需要使用更高的温度

```python
prompt = """
你是一位客户服务AI助手，你的任务是给你的客户发送一封电子邮件回复。
感谢客户的评论，如果情感是积极或中性，感谢他们的评论；
如果情感是消极的，道歉并建议他们联系客户服务。
确保使用评论中的具体细节。
用简明和专业的语气写信。
并签名为`AI客户代理`。

客户评论：
"""+review
"评论情感:"+sentiment
# +story
response = get_completion(prompt,temperature=0.7)
print(response)

结果1：
尊敬的客户，

感谢您的评论。我们非常抱歉您对我们的产品感到失望。我们非常重视您的反馈，以便我们可以改进我们的产品和服务。

我们深刻理解您对产品质量的关注，我们会尽最大努力确保我们的产品质量符合客户的期望。我们非常感谢您的建议，我们将会在未来的产品设计和制造中更加注重细节和质量。

如果您需要更多的帮助或有任何疑问，请随时联系我们的客户服务团队。我们将竭诚为您服务。

再次感谢您的反馈，我们期待为您提供更好的服务。

祝您一切顺利！

AI客户代理


结果2：
尊敬的客户，

感谢您的评论。我们非常抱歉您对我们的产品感到失望。我们非常重视您的反馈，我们将竭尽全力改进我们的产品和服务。

我们理解您对产品的期望，但是由于不同的显示器和光线条件，可能会导致颜色差异。我们会尽力确保我们的产品图片与实际产品相符，但是我们也需要考虑到这些因素。

如果您对我们的产品不满意，我们建议您联系我们的客户服务团队，我们将竭诚为您提供帮助和解决方案。我们希望能够为您提供更好的购物体验。

再次感谢您的反馈和支持。

祝您一切顺利！

AI客户代理
```



## 8.聊天机器人

如何为自己构建聊天机器人



system：有助于设置助手的行为和人设；为开发者提供了一种在不将请求本身作为对话的一部分的情况下引导助手并指导其回复的方式。

assistant：ChatGPT

user：me



与语言模型的每次对话都是独立的交互，所以必须提供当前对话中所有相关信息。



orderbot:自动化收集用户提示和助手响应



## 实操：连接API

```python
import openai

#api 密钥：openAI网站获取
openai.api_key = "xxx"

def get_completion(prompt,model = "gpt-3.5-turbo"):
    messages = [{"role":"user","content":prompt}]
    response = openai.ChatCompletion.create(
        model = model,
        messages = messages,
        temperature=0,
    )

    
    return response.choices[0].message["content"]

#提问
response = get_completion("请问你能帮我写英语作文吗")
print(response)
```



问题：直接开代理不行，不开代理不能访问

解决：

1.打开文件：anaconda3安装位置\Lib\site-packages\openai\api_requestor.py

2.添加代理：端口填写自己的代理端口

3.将proxies换成添加的proxy

![image-20230522132832096](C:\Users\发、\AppData\Roaming\Typora\typora-user-images\image-20230522132832096.png)



