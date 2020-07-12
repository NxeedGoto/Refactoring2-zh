<?xml version="1.0" encoding="utf-8" standalone="no"?><!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN"
  "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd"><html xmlns="http://www.w3.org/1999/xhtml" style="font-size:1.333rem;"><head>
  <link href="../Styles/style0002.css" rel="stylesheet" type="text/css"/>

  <title></title>
</head><body>
  <div style="page-break-after:always"></div><h1 id="nav_point_233">第9章　重新组织数据</h1>

  <p class="zw">数据结构在程序中扮演着重要的角色，所以毫不意外，我有一组重构手法专门用于数据结构的组织。将一个值用于多个不同的用途，这就是催生混乱和bug的温床。所以，一旦看见这样的情况，我就会用<span style="">拆分变量（240）</span>将不同的用途分开。和其他任何程序元素一样，给变量起个好名字不容易但又非常重要，所以我常会用到<span style="">变量改名（137）</span>。但有些多余的变量最好是彻底消除掉，比如通过<span style="">以查询取代派生变量（248）</span>。</p>

  <p class="zw">引用和值的混淆经常会造成问题，所以我会用<span style="">将引用对象改为值对象（252）</span>和<span style="">将值对象改为引用对象（256）</span>在两者之间切换。</p>

  <h2 id="nav_point_234">9.1　拆分变量（Split Variable）</h2>

  <p class="zw">曾用名：<span style="">移除对参数的赋值</span>（Remove Assignments to Parameters）</p>

  <p class="zw">曾用名：<span style="">分解临时变量</span>（Split Temp）</p>

  <p class="图"><img alt="" src="../Images/image00341.jpeg" style="width: 40%" width="40%"/></p>
  <pre class="代码无行号"><code>let temp = 2 * (height + width); 
console.log(temp);
temp = height * width; 
console.log(temp);</code></pre>

  <p class="图"><img alt="图像说明文字" src="../Images/image00342.jpeg" style="width: 7%" width="7%"/></p>
  <pre class="代码无行号"><code>const perimeter = 2 * (height + width); 
console.log(perimeter);
const area = height * width; 
console.log(area);</code></pre>

  <h3 class="sigil_not_in_toc" id="nav_point_235">动机</h3>

  <p class="zw">变量有各种不同的用途，其中某些用途会很自然地导致临时变量被多次赋值。“循环变量”和“结果收集变量”就是两个典型例子：循环变量（loop variable）会随循环的每次运行而改变（例如<code>for（let i=0; i&lt;10; i++）</code>语句中的<code>i</code>）；结果收集变量（collecting variable）负责将“通过整个函数的运算”而构成的某个值收集起来。</p>

  <p class="zw">除了这两种情况，还有很多变量用于保存一段冗长代码的运算结果，以便稍后使用。这种变量应该只被赋值一次。如果它们被赋值超过一次，就意味它们在函数中承担了一个以上的责任。如果变量承担多个责任，它就应该被替换（分解）为多个变量，每个变量只承担一个责任。同一个变量承担两件不同的事情，会令代码阅读者糊涂。</p>

  <h3 class="sigil_not_in_toc" id="nav_point_236">做法</h3>

  <ul>
    <li class="第1级无序列表">在待分解变量的声明及其第一次被赋值处，修改其名称。</li>
  </ul>

  <blockquote>
    <p class="zw">如果稍后的赋值语句是“<code>i=i+</code>某表达式形式”，意味着这是一个结果收集变量，就不要分解它。结果收集变量常用于累加、字符串拼接、写入流或者向集合添加元素。</p>
  </blockquote>

  <ul>
    <li class="第1级无序列表">如果可能的话，将新的变量声明为不可修改。</li>

    <li class="第1级无序列表">以该变量的第二次赋值动作为界，修改此前对该变量的所有引用，让它们引用新变量。</li>

    <li class="第1级无序列表">测试。</li>

    <li class="第1级无序列表">重复上述过程。每次都在声明处对变量改名，并修改下次赋值之前的引用，直至到达最后一处赋值。</li>
  </ul>

  <h3 class="sigil_not_in_toc" id="nav_point_237">范例</h3>

  <p class="zw">下面范例中我要计算一个苏格兰布丁运动的距离。在起点处，静止的苏格兰布丁会受到一个初始力的作用而开始运动。一段时间后，第二个力作用于布丁，让它再次加速。根据牛顿第二定律，我可以这样计算布丁运动的距离：</p>
  <pre class="代码无行号"><code>function distanceTravelled (scenario, time) { 
　let result;
　let acc = scenario.primaryForce / scenario.mass; 
　let primaryTime = Math.min(time, scenario.delay); 
　result = 0.5 * acc * primaryTime * primaryTime; 
　let secondaryTime = time - scenario.delay;
　if (secondaryTime &gt; 0) {
　　let primaryVelocity = acc * scenario.delay;
　　acc = (scenario.primaryForce + scenario.secondaryForce) / scenario.mass;
　　result += primaryVelocity * secondaryTime + 0.5 * acc * secondaryTime * secondaryTime;
　}
　return result;
}</code></pre>

  <p class="zw">真是个丑陋的小东西。注意观察此例中的<code>acc</code>变量是如何被赋值两次的。<code>acc</code>变量有两个责任：第一是保存第一个力造成的初始加速度；第二是保存两个力共同造成的加速度。这就是我想要分解的东西。</p>

  <blockquote>
    <p class="zw">在尝试理解变量被如何使用时，如果编辑器能高亮显示一个符号（symbol）在函数内或文件内出现的所有位置，会相当便利。大部分现代编辑器都可以轻松做到这一点。</p>
  </blockquote>

  <p class="zw">首先，我在函数开始处修改这个变量的名称，并将新变量声明为<code>const</code>。接着，我把新变量声明之后、第二次赋值之前对<code>acc</code>变量的所有引用，全部改用新变量。最后，我在第二次赋值处重新声明<code>acc</code>变量：</p>
  <pre class="代码无行号"><code>function distanceTravelled (scenario, time) { 
　let result;
　const primaryAcceleration = scenario.primaryForce / scenario.mass; 
　let primaryTime = Math.min(time, scenario.delay);
　result = 0.5 * primaryAcceleration * primaryTime * primaryTime; 
　let secondaryTime = time - scenario.delay;
　if (secondaryTime &gt; 0) {
　　let primaryVelocity = primaryAcceleration * scenario.delay;
　　let acc = (scenario.primaryForce + scenario.secondaryForce) / scenario.mass;
　　result += primaryVelocity * secondaryTime + 0.5 * acc * secondaryTime * secondaryTime;
　}
　return result;
}</code></pre>

  <p class="zw">新变量的名称指出，它只承担原先<code>acc</code>变量的第一个责任。我将它声明为<code>const</code>，确保它只被赋值一次。然后，我在原先<code>acc</code>变量第二次被赋值处重新声明<code>acc</code>。现在，重新编译并测试，一切都应该没有问题。</p>

  <p class="zw">然后，我继续处理<code>acc</code>变量的第二次赋值。这次我把原先的变量完全删掉，代之以一个新变量。新变量的名称指出，它只承担原先<code>acc</code>变量的第二个责任：</p>
  <pre class="代码无行号"><code>function distanceTravelled (scenario, time) { 
　let result;
　const primaryAcceleration = scenario.primaryForce / scenario.mass;
　let primaryTime = Math.min(time, scenario.delay);
　result = 0.5 * primaryAcceleration * primaryTime * primaryTime; 
　let secondaryTime = time - scenario.delay;
　if (secondaryTime &gt; 0) {
　　let primaryVelocity = primaryAcceleration * scenario.delay;
　　const secondaryAcceleration = (scenario.primaryForce + scenario.secondaryForce) / scenario.mass; 
　　result += primaryVelocity * secondaryTime +
　　　0.5 * secondaryAcceleration * secondaryTime * secondaryTime;
　}
　return result;
}</code></pre>

  <p class="zw">现在，这段代码肯定可以让你想起更多其他重构手法。尽情享受吧。（我敢保证，这比吃苏格兰布丁强多了——你知道他们都在里面放了些什么东西吗？<span class="注释编号">1</span> ）</p>

  <h3 class="sigil_not_in_toc" id="nav_point_238">范例：对输入参数赋值</h3>

  <p class="zw">另一种情况是，变量是以输入参数的形式声明又在函数内部被再次赋值，此时也可以考虑拆分变量。例如，下列代码：</p>
  <pre class="代码无行号"><code>function discount (inputValue, quantity) {
　if (inputValue &gt; 50) inputValue = inputValue - 2; 
　if (quantity &gt; 100) inputValue = inputValue - 1; 
　return inputValue;
}</code></pre>

  <p class="zw">这里的<code>inputValue</code>有两个用途：它既是函数的输入，也负责把结果带回给调用方。（由于JavaScript的参数是按值传递的，所以函数内部对<code>inputValue</code>的修改不会影响调用方。）</p>

  <p class="zw">在这种情况下，我就会对<code>inputValue</code>变量做拆分。</p>
  <pre class="代码无行号"><code>function discount (originalInputValue, quantity) { 
　let inputValue = originalInputValue;
　if (inputValue &gt; 50) inputValue = inputValue - 2;
　if (quantity &gt; 100) inputValue = inputValue - 1; 
　return inputValue;
}</code></pre>

  <p class="zw">然后用<span style="">变量改名（137）</span>给两个变量换上更好的名字。</p>
  <pre class="代码无行号"><code>function discount (inputValue, quantity) { 
　let result = inputValue;
　if (inputValue &gt; 50) result = result - 2;
　if (quantity &gt; 100) result = result - 1; 
　return result;
}</code></pre>

  <p class="zw">我修改了第二行代码，把<code>inputValue</code>作为判断条件的基准数据。虽说这里用<code>inputValue</code>还是<code>result</code>效果都一样，但在我看来，这行代码的含义是“根据原始输入值做判断，然后修改结果值”，而不是“根据当前结果值做判断”——尽管两者的效果恰好一样。</p>

  <p class="注释内容"><span class="注释编号下">1</span>苏格兰布丁（haggis）是一种苏格兰菜，把羊心等内脏装在羊胃里煮成。由于它被羊胃包成一个球体，因此可以像球一样踢来踢去，这就是本例的由来。“把羊心装在羊胃里煮成……”，呃，有些人难免对这道菜恶心，Martin Fowler想必是其中之一。——译者注</p>

  <h2 id="nav_point_239">9.2　字段改名（Rename Field）</h2>

  <p class="图"><img alt="" src="../Images/image00343.jpeg" style="width: 40%" width="40%"/></p>
  <pre class="代码无行号"><code>class Organization { 
  get name() {...}
}</code></pre>

  <p class="图"><img alt="图像说明文字" src="../Images/image00344.jpeg" style="width: 7%" width="7%"/></p>
  <pre class="代码无行号"><code>class Organization { 
  get title() {...}
}</code></pre>

  <h3 class="sigil_not_in_toc" id="nav_point_240">动机</h3>

  <p class="zw">命名很重要，对于程序中广泛使用的记录结构，其中字段的命名格外重要。数据结构对于帮助阅读者理解特别重要。多年以前，Fred Brooks就说过：“只给我看你的工作流程却隐藏表单，我将仍然一头雾水。但是如果你给我展示表单，或许不需要流程图，就能柳暗花明。”现在已经不太有人画流程图了，不过道理还是一样的。数据结构是理解程序行为的关键。</p>

  <p class="zw">既然数据结构如此重要，就很有必要保持它们的整洁。一如既往地，我在一个软件上做的工作越多，对数据的理解就越深，所以很有必要把我加深的理解融入程序中。</p>

  <p class="zw">记录结构中的字段可能需要改名，类的字段也一样。在类的使用者看来，取值和设值函数就等于是字段。对这些函数的改名，跟裸记录结构的字段改名一样重要。</p>

  <h3 class="sigil_not_in_toc" id="nav_point_241">做法</h3>

  <ul>
    <li class="第1级无序列表">如果记录的作用域较小，可以直接修改所有该字段的代码，然后测试。后面的步骤就都不需要了。</li>

    <li class="第1级无序列表">如果记录还未封装，请先使用<span style="">封装记录（162）</span>。</li>

    <li class="第1级无序列表">在对象内部对私有字段改名，对应调整内部访问该字段的函数。</li>

    <li class="第1级无序列表">测试。</li>

    <li class="第1级无序列表">如果构造函数的参数用了旧的字段名，运用<span style="">改变函数声明（124）</span>将其改名。</li>

    <li class="第1级无序列表">运用<span style="">函数改名</span>（124）给访问函数改名。</li>
  </ul>

  <h3 class="sigil_not_in_toc" id="nav_point_242">范例：给字段改名</h3>

  <p class="zw">我们从一个常量开始。</p>
  <pre class="代码无行号"><code>const organization = {name: "Acme Gooseberries", country: "GB"};</code></pre>

  <p class="zw">我想把<code>name</code>改名为<code>title</code>。这个对象被很多地方使用，有些代码会更新<code>name</code>字段。所以我首先要用<span style="">封装记录（162）</span>把这个记录封装起来。</p>
  <pre class="代码无行号"><code>class Organization { 
　constructor(data) {
　　this._name = data.name; 
　　this._country = data.country;
　}
　get name()　{return this._name;}
　set name(aString) {this._name = aString;} 
　get country()　{return this._country;}
　set country(aCountryCode) {this._country = aCountryCode;}
}

const organization = new Organization({name: "Acme Gooseberries", country: "GB"});</code></pre>

  <p class="zw">现在，记录结构已经被封装成类。在对字段改名时，有4个地方需要留意：取值函数、设值函数、构造函数以及内部数据结构。这听起来似乎是增加了重构的工作量，但现在我可以分别小步修改这4处，而不必一次修改所有地方，所以其实是降低了重构的难度。小步修改就意味着每一步出错的可能性大大减小，因此会省掉很多工作量——如果我从不犯错，小步前进不会节省工作量；但“从不犯错”这样的梦，我很久以前就已经不做了。</p>

  <p class="zw">由于已经把输入数据复制到内部数据结构中，现在我需要将这两个数据结构区分开，以便各自单独处理。我可以另外定义一个字段，修改构造函数和访问函数，令其使用新字段。</p>

  <h5 class="sigil_not_in_toc">class Organization...</h5>
  <pre class="代码无行号"><code>class Organization { 
　constructor(data) {
　　this._title = data.name; 
　　this._country = data.country;
　}
　get name()　{return this._title;}
　set name(aString) {this._title = aString;} 
　get country()　{return this._country;}
　set country(aCountryCode) {this._country = aCountryCode;}
}</code></pre>

  <p class="zw">接下来我就可以在构造函数中使用<code>title</code>字段。</p>

  <h5 class="sigil_not_in_toc">class Organization...</h5>
  <pre class="代码无行号"><code>class Organization { 
　constructor(data) {
　　this._title = (data.title !== undefined) ? data.title : data.name;
　　this._country = data.country;
　}
　get name()　　　　{return this._title;}
　set name(aString) {this._title = aString;} 
　get country()　　 {return this._country;}
　set country(aCountryCode) {this._country = aCountryCode;}
}</code></pre>

  <p class="zw">现在，构造函数的调用者既可以使用<code>name</code>也可以使用<code>title</code>（后者的优先级更高）。我会逐一查看所有调用构造函数的地方，将它们改为使用新名字。</p>
  <pre class="代码无行号"><code>const organization = new Organization({title: "Acme Gooseberries", country: "GB"});</code></pre>

  <p class="zw">全部修改完成后，就可以在构造函数中去掉对<code>name</code>的支持，只使用<code>title</code>。</p>

  <h5 class="sigil_not_in_toc">class Organization...</h5>
  <pre class="代码无行号"><code>class Organization { 
　constructor(data) {
　　this._title = data.title; 
　　this._country = data.country;
　}
　get name()　　　　{return this._title;}
　set name(aString) {this._title = aString;} 
　get country()　　 {return this._country;}
　set country(aCountryCode) {this._country = aCountryCode;}
}</code></pre>

  <p class="zw">现在构造函数和内部数据结构都已经在使用新名字了，接下来我就可以给访问函数改名。这一步很简单，只要对每个访问函数运用<span style="">函数改名</span>（124）就行了。</p>

  <h5 class="sigil_not_in_toc">class Organization...</h5>
  <pre class="代码无行号"><code>class Organization { 
　constructor(data) {
　　this._title = data.title; 
　　this._country = data.country;
　}
　get title()  {return this._title;}
　set title(aString) {this._title = aString;} 
　get country()  　{return this._country;}
　set country(aCountryCode) {this._country = aCountryCode;}
}</code></pre>

  <p class="zw">上面展示的重构过程，是本重构手法最重量级的做法，只有对广泛使用的数据结构才用得上。如果该数据结构只在较小的范围（例如单个函数）中用到，我可能可以一步到位地完成所有改名动作，不需要提前做封装。何时需要用上全套重量级做法，这由你自己判断——如果在重构过程中破坏了测试，我通常会视之为一个信号，说明我需要改用更渐进的方式来重构。</p>

  <p class="zw">有些编程语言允许将数据结构声明为不可变。在这种情况下，我可以把旧字段的值复制到新名字下，逐一修改使用方代码，然后删除旧字段。对于可变的数据结构，重复数据会招致灾难；而不可变的数据结构则没有这些麻烦。这也是大家愿意使用不可变数据的原因。</p>

  <h2 id="nav_point_243">9.3　以查询取代派生变量（Replace Derived Variable with Query）</h2>

  <p class="图"><img alt="" src="../Images/image00345.jpeg" style="width: 40%" width="40%"/></p>
  <pre class="代码无行号"><code>get discountedTotal() {return this._discountedTotal;} 
set discount(aNumber) {
　const old = this._discount; 
　this._discount = aNumber; 
　this._discountedTotal += old - aNumber;
}</code></pre>

  <p class="图"><img alt="图像说明文字" src="../Images/image00346.jpeg" style="width: 7%" width="7%"/></p>
  <pre class="代码无行号"><code>get discountedTotal() {return this._baseTotal - this._discount;} 
set discount(aNumber) {this._discount = aNumber;}</code></pre>

  <h3 class="sigil_not_in_toc" id="nav_point_244">动机</h3>

  <p class="zw">可变数据是软件中最大的错误源头之一。对数据的修改常常导致代码的各个部分以丑陋的形式互相耦合：在一处修改数据，却在另一处造成难以发现的破坏。很多时候，完全去掉可变数据并不现实，但我还是强烈建议：尽量把可变数据的作用域限制在最小范围。</p>

  <p class="zw">有些变量其实可以很容易地随时计算出来。如果能去掉这些变量，也算朝着消除可变性的方向迈出了一大步。计算常能更清晰地表达数据的含义，而且也避免了“源数据修改时忘了更新派生变量”的错误。</p>

  <p class="zw">有一种合理的例外情况：如果计算的源数据是不可变的，并且我们可以强制要求计算的结果也是不可变的，那么就不必重构消除计算得到的派生变量。因此，“根据源数据生成新数据结构”的变换操作可以保持不变，即便我们可以将其替换为计算操作。实际上，这是两种不同的编程风格：一种是对象风格，把一系列计算得出的属性包装在数据结构中；另一种是函数风格，将一个数据结构变换为另一个数据结构。如果源数据会被修改，而你必须负责管理派生数据结构的整个生命周期，那么对象风格显然更好。但如果源数据不可变，或者派生数据用过即弃，那么两种风格都可行。</p>

  <h3 class="sigil_not_in_toc" id="nav_point_245">做法</h3>

  <ul>
    <li class="第1级无序列表">识别出所有对变量做更新的地方。如有必要，用<span style="">拆分变量（240）</span>分割各个更新点。</li>

    <li class="第1级无序列表">新建一个函数，用于计算该变量的值。</li>

    <li class="第1级无序列表">用<span style="">引入断言（302）</span>断言该变量和计算函数始终给出同样的值。</li>
  </ul>

  <blockquote>
    <p class="zw">如有必要，用<span style="">封装变量（132）</span>将这个断言封装起来。</p>
  </blockquote>

  <ul>
    <li class="第1级无序列表">测试。</li>

    <li class="第1级无序列表">修改读取该变量的代码，令其调用新建的函数。</li>

    <li class="第1级无序列表">测试。</li>

    <li class="第1级无序列表">用<span style="">移除死代码（237）</span>去掉变量的声明和赋值。</li>
  </ul>

  <h3 class="sigil_not_in_toc" id="nav_point_246">范例</h3>

  <p class="zw">下面这个例子虽小，却完美展示了代码的丑陋。</p>

  <h5 class="sigil_not_in_toc">class ProductionPlan...</h5>
  <pre class="代码无行号"><code>get production() {return this._production;} 
applyAdjustment(anAdjustment) {
　this._adjustments.push(anAdjustment); 
　this._production += anAdjustment.amount;
}</code></pre>

  <p class="zw">丑与不丑，全在观者。我看到的丑陋之处是重复——不是常见的代码重复，而是数据的重复。如果我要对生产计划（production plan）做调整（adjustment），不光要把调整的信息保存下来，还要根据调整信息修改一个累计值——后者完全可以即时计算，而不必每次更新。</p>

  <p class="zw">但我是个谨慎的人。“可以即时计算”只是我的猜想——我可以用<span style="">引入断言（302）</span>来验证这个猜想。</p>

  <h5 class="sigil_not_in_toc">class ProductionPlan...</h5>
  <pre class="代码无行号"><code>get production() {
　assert(this._production === this.calculatedProduction);
　return this._production;
}

get calculatedProduction() { 
　return this._adjustments
　　.reduce((sum, a) =&gt; sum + a.amount, 0);
}</code></pre>

  <p class="zw">放上这个断言之后，我会运行测试。如果断言没有失败，我就可以不再返回该字段，改为返回即时计算的结果。</p>

  <h5 class="sigil_not_in_toc">class ProductionPlan...</h5>
  <pre class="代码无行号"><code>get production() {
  <del>assert(this._production === this.calculatedProduction);</del>
  return this.calculatedProduction;
}</code></pre>

  <p class="zw">然后用<span style="">内联函数（115）</span>把计算逻辑内联到<code>production</code>函数内。</p>

  <h5 class="sigil_not_in_toc">class ProductionPlan...</h5>
  <pre class="代码无行号"><code>get production() {
  return this._adjustments
    .reduce((sum, a) =&gt; sum + a.amount, 0);
}</code></pre>

  <p class="zw">再用<span style="">移除死代码（237）</span>扫清使用旧变量的地方。</p>

  <h5 class="sigil_not_in_toc">class ProductionPlan...</h5>
  <pre class="代码无行号"><code>applyAdjustment(anAdjustment) { 
  this._adjustments.push(anAdjustment); 
  <del>this._production += anAdjustment.amount;</del>
}</code></pre>

  <h3 class="sigil_not_in_toc" id="nav_point_247">范例：不止一个数据来源</h3>

  <p class="zw">上面的例子处理得轻松愉快，因为<code>production</code>的值很明显只有一个来源。但有时候，累计值会受到多个数据来源的影响。</p>

  <h5 class="sigil_not_in_toc">class ProductionPlan...</h5>
  <pre class="代码无行号"><code>constructor (production) { 
　this._production = production; 
　this._adjustments = [];
}
get production() {return this._production;} 
applyAdjustment(anAdjustment) {
　this._adjustments.push(anAdjustment);
　this._production += anAdjustment.amount;
}</code></pre>

  <p class="zw">如果照上面的方式运用<span style="">引入断言（302）</span>，只要<code>production</code>的初始值不为0，断言就会失败。</p>

  <p class="zw">不过我还是可以替换派生数据，只不过必须先运用<span style="">拆分变量（240）</span>。</p>
  <pre class="代码无行号"><code>constructor (production) { 
　this._initialProduction = production; 
　this._productionAccumulator = 0; 
　this._adjustments = [];
}
get production() {
　return this._initialProduction + this._productionAccumulator;
}</code></pre>

  <p class="zw">现在我就可以使用<span style="">引入断言（302）。</span></p>

  <h5 class="sigil_not_in_toc">class ProductionPlan...</h5>
  <pre class="代码无行号"><code>get production() {
　assert(this._productionAccumulator === this.calculatedProductionAccumulator);
　return this._initialProduction + this._productionAccumulator;
}

get calculatedProductionAccumulator() { 
　return this._adjustments
　　.reduce((sum, a) =&gt; sum + a.amount, 0);
}</code></pre>

  <p class="zw">接下来的步骤就跟前一个范例一样了。不过我会更愿意保留<code>calculatedProduction Accumulator</code>这个属性，而不把它内联消去。</p>

  <h2 id="nav_point_248">9.4　将引用对象改为值对象（Change Reference to Value）</h2>

  <p class="zw">反向重构：<span style="">将值对象改为引用对象（256）</span></p>

  <p class="图"><img alt="" src="../Images/image00347.jpeg" style="width: 40%" width="40%"/></p>
  <pre class="代码无行号"><code>class Product {
  applyDiscount(arg) {this._price.amount -= arg;}</code></pre>

  <p class="图"><img alt="图像说明文字" src="../Images/image00348.jpeg" style="width: 7%" width="7%"/></p>
  <pre class="代码无行号"><code>class Product { 
  applyDiscount(arg) {
    this._price = new Money(this._price.amount - arg, this._price.currency);
  }</code></pre>

  <h3 class="sigil_not_in_toc" id="nav_point_249">动机</h3>

  <p class="zw">在把一个对象（或数据结构）嵌入另一个对象时，位于内部的这个对象可以被视为引用对象，也可以被视为值对象。两者最明显的差异在于如何更新内部对象的属性：如果将内部对象视为引用对象，在更新其属性时，我会保留原对象不动，更新内部对象的属性；如果将其视为值对象，我就会替换整个内部对象，新换上的对象会有我想要的属性值。</p>

  <p class="zw">如果把一个字段视为值对象，我可以把内部对象的类也变成值对象[mf-vo]。值对象通常更容易理解，主要因为它们是不可变的。一般说来，不可变的数据结构处理起来更容易。我可以放心地把不可变的数据值传给程序的其他部分，而不必担心对象中包装的数据被偷偷修改。我可以在程序各处复制值对象，而不必操心维护内存链接。值对象在分布式系统和并发系统中尤为有用。</p>

  <p class="zw">值对象和引用对象的区别也告诉我，何时不应该使用本重构手法。如果我想在几个对象之间共享一个对象，以便几个对象都能看见对共享对象的修改，那么这个共享的对象就应该是引用。</p>

  <h3 class="sigil_not_in_toc" id="nav_point_250">做法</h3>

  <ul>
    <li class="第1级无序列表">检查重构目标是否为不可变对象，或者是否可修改为不可变对象。</li>

    <li class="第1级无序列表">用<span style="">移除设值函数（331）</span>逐一去掉所有设值函数。</li>

    <li class="第1级无序列表">提供一个基于值的相等性判断函数，在其中使用值对象的字段。</li>
  </ul>

  <blockquote>
    <p class="zw">大多数编程语言都提供了可覆写的相等性判断函数。通常你还必须同时覆写生成散列码的函数。</p>
  </blockquote>

  <h3 class="sigil_not_in_toc" id="nav_point_251">范例</h3>

  <p class="zw">设想一个代表“人”的<code>Person</code>类，其中包含一个代表“电话号码”的<code>Telephone Number</code>对象。</p>

  <h5 class="sigil_not_in_toc">class Person...</h5>

  <p class="zw">constructor() {</p>
  <pre class="代码无行号"><code>constructor() {
　this._telephoneNumber = new TelephoneNumber();
}

get officeAreaCode()　　{return this._telephoneNumber.areaCode;} 
set officeAreaCode(arg) {this._telephoneNumber.areaCode = arg;} 
get officeNumber()　　{return this._telephoneNumber.number;}
set officeNumber(arg) {this._telephoneNumber.number = arg;}</code></pre>

  <h5 class="sigil_not_in_toc">class TelephoneNumber...</h5>
  <pre class="代码无行号"><code>get areaCode()    {return this._areaCode;} 
set areaCode(arg) {this._areaCode = arg;}

get number()    {return this._number;} 
set number(arg) {this._number = arg;}</code></pre>

  <p class="zw">代码的当前状态是<span style="">提炼类（182）</span>留下的结果：从前拥有电话号码信息的<code>Person</code>类仍然有一些函数在修改新对象的属性。趁着还只有一个指向新类的引用，现在是时候使用<span style="">将引用对象改为值对象</span>将其变成值对象。</p>

  <p class="zw">我需要做的第一件事是把<code>TelephoneNumber</code>类变成不可变的。对它的字段运用<span style="">移除设值函数（331）</span>。<span style="">移除设值函数（331）</span>的第一步是，用<span style="">改变函数声明（124）</span>把这两个字段的初始值加到构造函数中，并迫使构造函数调用设值函数。</p>

  <h5 class="sigil_not_in_toc">class TelephoneNumber...</h5>
  <pre class="代码无行号"><code>constructor(areaCode, number) { 
  this._areaCode = areaCode; 
  this._number = number;
}</code></pre>

  <p class="zw">然后我会逐一查看设值函数的调用者，并将其改为重新赋值整个对象。先从“地区代码”（area code）开始。</p>

  <h5 class="sigil_not_in_toc">class Person...</h5>
  <pre class="代码无行号"><code>get officeAreaCode()    {return this._telephoneNumber.areaCode;} 
set officeAreaCode(arg) {
　this._telephoneNumber = new TelephoneNumber(arg, this.officeNumber);
}
get officeNumber()　  {return this._telephoneNumber.number;} 
set officeNumber(arg) {this._telephoneNumber.number = arg;}</code></pre>

  <p class="zw">对于其他字段，重复上述步骤。</p>

  <h5 class="sigil_not_in_toc">class Person...</h5>
  <pre class="代码无行号"><code>get officeAreaCode()    {return this._telephoneNumber.areaCode;} 
set officeAreaCode(arg) {
  this._telephoneNumber = new TelephoneNumber(arg, this.officeNumber);
}
get officeNumber()    {return this._telephoneNumber.number;} 
set officeNumber(arg) {
　this._telephoneNumber = new TelephoneNumber(this.officeAreaCode, arg);
}</code></pre>

  <p class="zw">现在，<code>TelephoneNumber</code>已经是不可变的类，可以将其变成真正的值对象了。是不是真正的值对象，要看是否基于值判断相等性。在这个领域中，JavaScript做得不好：语言和核心库都不支持将“基于引用的相等性判断”换成“基于值的相等性判断”。我唯一能做的就是创建自己的<code>equals</code>函数。</p>

  <h5 class="sigil_not_in_toc">class TelephoneNumber...</h5>
  <pre class="代码无行号"><code>equals(other) {
　if (!(other instanceof TelephoneNumber)) return false; 
　return this.areaCode === other.areaCode &amp;&amp;
　　this.number === other.number;
}</code></pre>

  <p class="zw">对其进行测试很重要：</p>
  <pre class="代码无行号"><code>it('telephone equals', function() {
　assert(　　　　new TelephoneNumber("312", "555-0142")
　　　　 .equals(new TelephoneNumber("312", "555-0142")));
});</code></pre>

  <p class="zw"><span style="">这段测试代码用了不寻常的格式，是为了帮助读者一眼看出上下两次构造函数调用完全一样。</span></p>

  <p class="zw">我在这个测试中创建了两个各自独立的对象，并验证它们相等。</p>

  <blockquote>
    <p class="zw">在大多数面向对象语言中，内置的相等性判断方法可以被覆写为基于值的相等性判断。在Ruby中，我可以覆写<code>==</code>运算符；在Java中，我可以覆写<code>Object.equals()</code>方法。在覆写相等性判断的同时，我通常还需要覆写生成散列码的方法（例如Java中的<code>Object.hashCode()</code>方法），以确保用到散列码的集合在使用值对象时一切正常。</p>
  </blockquote>

  <p class="zw">如果有多个客户端使用了<code>TelephoneNumber</code>对象，重构的过程还是一样，只是在运用<span style="">移除设值函数（331）</span>时要修改多处客户端代码。另外，有必要添加几个测试，检查电话号码不相等以及与非电话号码和<code>null</code>值比较相等性等情况。</p>

  <h2 id="nav_point_252">9.5　将值对象改为引用对象（Change Value to Reference）</h2>

  <p class="zw">反向重构：<span style="">将引用对象改为值对象（252）</span></p>

  <p class="图"><img alt="" src="../Images/image00349.jpeg" style="width: 40%" width="40%"/></p>
  <pre class="代码无行号"><code>let customer = new Customer(customerData);</code></pre>

  <p class="图"><img alt="图像说明文字" src="../Images/image00350.jpeg" style="width: 7%" width="7%"/></p>
  <pre class="代码无行号"><code>let customer = customerRepository.get(customerData.id);</code></pre>

  <h3 class="sigil_not_in_toc" id="nav_point_253">动机</h3>

  <p class="zw">一个数据结构中可能包含多个记录，而这些记录都关联到同一个逻辑数据结构。例如，我可能会读取一系列订单数据，其中有多条订单属于同一个顾客。遇到这样的共享关系时，既可以把顾客信息作为值对象来看待，也可以将其视为引用对象。如果将其视为值对象，那么每份订单数据中都会复制顾客的数据；而如果将其视为引用对象，对于一个顾客，就只有一份数据结构，会有多个订单与之关联。</p>

  <p class="zw">如果顾客数据永远不修改，那么两种处理方式都合理。把同一份数据复制多次可能会造成一点困扰，但这种情况也很常见，不会造成太大问题。过多的数据复制有可能会造成内存占用的问题，但就跟所有性能问题一样，这种情况并不常见。</p>

  <p class="zw">如果共享的数据需要更新，将其复制多份的做法就会遇到巨大的困难。此时我必须找到所有的副本，更新所有对象。只要漏掉一个副本没有更新，就会遭遇麻烦的数据不一致。这种情况下，可以考虑将多份数据副本变成单一的引用，这样对顾客数据的修改就会立即反映在该顾客的所有订单中。</p>

  <p class="zw">把值对象改为引用对象会带来一个结果：对于一个客观实体，只有一个代表它的对象。这通常意味着我会需要某种形式的仓库，在仓库中可以找到所有这些实体对象。只为每个实体创建一次对象，以后始终从仓库中获取该对象。</p>

  <h3 class="sigil_not_in_toc" id="nav_point_254">做法</h3>

  <ul>
    <li class="第1级无序列表">为相关对象创建一个仓库（如果还没有这样一个仓库的话）。</li>

    <li class="第1级无序列表">确保构造函数有办法找到关联对象的正确实例。</li>

    <li class="第1级无序列表">修改宿主对象的构造函数，令其从仓库中获取关联对象。每次修改后执行测试。</li>
  </ul>

  <h3 class="sigil_not_in_toc" id="nav_point_255">范例</h3>

  <p class="zw">我将从一个代表“订单”的<code>Order</code>类开始，其实例对象可从一个JSON文件创建。用来创建订单的数据中有一个顾客（customer）ID，我们用它来进一步创建<code>Customer</code>对象。</p>

  <h5 class="sigil_not_in_toc">class Order...</h5>
  <pre class="代码无行号"><code>constructor(data) { 
  this._number = data.number;
  this._customer = new Customer(data.customer);
  // load other data
}
get customer() {return this._customer;}</code></pre>

  <h5 class="sigil_not_in_toc">class Customer...</h5>
  <pre class="代码无行号"><code>constructor(id) {
  this._id = id;
}
get id() {return this._id;}</code></pre>

  <p class="zw">以这种方式创建的<code>Customer</code>对象是值对象。如果有5个订单都属于ID为<code>123</code>的顾客，就会有5个各自独立的<code>Customer</code>对象。对其中一个所做的修改，不会反映在其他几个对象身上。如果我想增强<code>Customer</code>对象，例如从客户服务获取到了更多关于顾客的信息，我必须用同样的数据更新所有5个对象。重复的对象总是会让我紧张——用多个对象代表同一个实体（例如一名顾客），这会招致混乱。如果<code>Customer</code>对象是可变的，问题就更加严重，因为各个对象之间的数据可能不一致。</p>

  <p class="zw">如果我想每次都使用同一个<code>Customer</code>对象，那么就需要有一个地方存储这个对象。每个应用程序中，存储实体的地方会各有不同，在最简单的情况下，我会使用一个仓库对象<span style="">[mf-repos]</span>。</p>
  <pre class="代码无行号"><code>let _repositoryData;

export function initialize() {
　_repositoryData = {};
　_repositoryData.customers = new Map();
}

export function registerCustomer(id) {
　if (! _repositoryData.customers.has(id))
　　_repositoryData.customers.set(id, new Customer(id)); 
　return findCustomer(id);
}

export function findCustomer(id) {
　return _repositoryData.customers.get(id);
}</code></pre>

  <p class="zw">仓库对象允许根据ID注册顾客，并且对于一个ID只会创建一个<code>Customer</code>对象。有了仓库对象，我就可以修改<code>Order</code>对象的构造函数来使用它。</p>

  <p class="zw">在使用本重构手法时，可能仓库对象已经存在了，那么就可以直接使用它。</p>

  <p class="zw">下一步是要弄清楚，<code>Order</code>的构造函数如何获得正确的<code>Customer</code>对象。在这个例子里，这一步很简单，因为输入数据流中已经包含了顾客的ID。</p>

  <h5 class="sigil_not_in_toc">class Order...</h5>
  <pre class="代码无行号"><code>constructor(data) { 
　this._number = data.number;
　this._customer = registerCustomer(data.customer);
　// load other data
}
get customer() {return this._customer;}</code></pre>

  <p class="zw">现在，如果我在一条订单中修改了顾客信息，就会同步反映在该顾客拥有的所有订单中。</p>

  <p class="zw">在这个例子里，我在第一个引用该顾客信息的<code>Order</code>对象中新建了<code>Customer</code>对象。另一个常见的做法是：首先获取一份包含所有<code>Customer</code>对象的列表，将其填入仓库对象，然后在读取<code>Order</code>对象时关联到对应的<code>Customer</code>对象。如果这样做，那么<code>Order</code>对象包含的顾客ID必须指向一个仓库中已有的<code>Customer</code>对象，否则就表示程序中有错误。</p>

  <p class="zw">上面的代码还有一个问题：构造函数与一个全局的仓库对象耦合。全局对象必须小心对待：它们就像强力的药物，少用一点儿大有益处，用过量就是毒药。如果想解决这个问题，可以将仓库对象作为参数传递给构造函数。</p>

  <p class="zw"><br style="page-break-after:always"/><div style="page-break-after:always"></div></p>
</body></html>