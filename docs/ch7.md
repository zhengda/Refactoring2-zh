<?xml version="1.0" encoding="utf-8" standalone="no"?><!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN"
  "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd"><html xmlns="http://www.w3.org/1999/xhtml" style="font-size:1.333rem;"><head>
  <link href="../Styles/style0002.css" rel="stylesheet" type="text/css"/>

  <title></title>
</head><body>
  <div style="page-break-after:always"></div><h1 id="nav_point_155">第7章　封装</h1>

  <p class="zw">分解模块时最重要的标准，也许就是识别出那些模块应该对外界隐藏的小秘密了[Parnas]。数据结构无疑是最常见的一种秘密，我可以用<span style="">封装记录（162）</span>或<span style="">封装集合（170）</span>手法来隐藏它们的细节。即便是基本类型的数据，也能通过<span style="">以对象取代基本类型（174）</span>进行封装——这样做后续所带来的巨大收益通常令人惊喜。另一项经常在重构时挡道的是临时变量，我需要确保它们的计算次序正确，还得保证其他需要它们的地方能获得其值。这里<span style="">以查询取代临时变量（178）</span>手法可以帮上大忙，特别是在分解一个过长的函数时。</p>

  <p class="zw">类是为隐藏信息而生的。在第6章中，我已经介绍了使用<span style="">函数组合成类（144）</span>手法来形成类的办法。此外，一般的提炼/内联操作对类也适用，见<span style="">提炼类（182）</span>和<span style="">内联类（186）</span>。</p>

  <p class="zw">除了类的内部细节，使用<span style="">隐藏委托关系（189）</span>隐藏类之间的关联关系通常也很有帮助。但过多隐藏也会导致冗余的中间接口，此时我就需要它的反向重构——<span style="">移除中间人（192）</span>。</p>

  <p class="zw">类与模块已然是施行封装的最大实体了，但小一点的函数对于封装实现细节也有所裨益。有时候，我可能需要将一个算法完全替换掉，这时我可以用<span style="">提炼函数（106）</span>将算法包装到函数中，然后使用<span style="">替换算法（195）</span>。</p>

  <h2 id="nav_point_156">7.1　封装记录（Encapsulate Record）</h2>

  <p class="zw">曾用名：<span style="">以数据类取代记录（Replace Record with Data Class）</span></p>

  <p class="图"><img alt="" src="../Images/image00304.jpeg" style="width: 40%" width="40%"/></p>
  <pre class="代码无行号"><code>organization = {name: "Acme Gooseberries", country: "GB"};</code></pre>

  <p class="图"><img alt="图像说明文字" src="../Images/image00305.jpeg" style="width: 7%" width="7%"/></p>
  <pre class="代码无行号"><code>class Organization { 
　constructor(data) {
　　this._name = data.name; 
　　this._country = data.country;
　}
　get name()    {return this._name;} 
　set name(arg) {this._name = arg;}
　get country()    {return this._country;} 
　set country(arg) {this._country = arg;}
}</code></pre>

  <h3 class="sigil_not_in_toc" id="nav_point_157">动机</h3>

  <p class="zw">记录型结构是多数编程语言提供的一种常见特性。它们能直观地组织起存在关联的数据，让我可以将数据作为有意义的单元传递，而不仅是一堆数据的拼凑。但简单的记录型结构也有缺陷，最恼人的一点是，它强迫我清晰地区分“记录中存储的数据”和“通过计算得到的数据”。假使我要描述一个整数闭区间，我可以用<code>{start: 1, end: 5}</code>描述，或者用<code>{start: 1, length: 5}</code>（甚至还能用<code>{end: 5, length: 5}</code>，如果我想露两手华丽的编程技巧的话）。但不论如何存储，这3个值都是我想知道的，即区间的起点（<code>start</code>）和终点（<code>end</code>），以及区间的长度（<code>length</code>）。</p>

  <p class="zw">这就是对于可变数据，我总是更偏爱使用类对象而非记录的原因。对象可以隐藏结构的细节，仅为这3个值提供对应的方法。该对象的用户不必追究存储的细节和计算的过程。同时，这种封装还有助于字段的改名：我可以重新命名字段，但同时提供新老字段名的访问方法，这样我就可以渐进地修改调用方，直到替换全部完成。</p>

  <p class="zw">注意，我所说的偏爱对象，是对可变数据而言。如果数据不可变，我大可直接将这3个值保存在记录里，需要做数据变换时增加一个填充步骤即可。重命名记录也一样简单，你可以复制一个字段并逐步替换引用点。</p>

  <p class="zw">记录型结构可以有两种类型：一种需要声明合法的字段名字，另一种可以随便用任何字段名字。后者常由语言库本身实现，并通过类的形式提供出来，这些类称为散列（hash）、映射（map）、散列映射（hashmap）、字典（dictionary）或关联数组（associative array）等。很多编程语言都提供了方便的语法来创建这类记录，这使得它们在各种编程场景下都能大展身手。但使用这类结构也有缺陷，那就是一条记录上持有什么字段往往不够直观。比如说，如果我想知道记录里维护的字段究竟是起点/终点还是起点/长度，就只有查看它的创建点和使用点，除此以外别无他法。若这种记录只在程序的一个小范围里使用，那问题还不大，但若其使用范围变宽，“数据结构不直观”这个问题就会造成更多困扰。我可以重构它，使其变得更直观——但如果真需要这样做，那还不如使用类来得直接。</p>

  <p class="zw">程序中间常常需要互相传递嵌套的列表（list）或散列映射结构，这些数据结构后续经常需要被序列化成JSON或XML。这样的嵌套结构同样值得封装，这样，如果后续其结构需要变更或者需要修改记录内的值，封装能够帮我更好地应对变化。</p>

  <h3 class="sigil_not_in_toc" id="nav_point_158">做法</h3>

  <ul>
    <li class="第1级无序列表">对持有记录的变量使用<span style="">封装变量（132）</span>，将其封装到一个函数中。</li>
  </ul>

  <blockquote>
    <p class="zw">记得为这个函数取一个容易搜索的名字。</p>
  </blockquote>

  <ul>
    <li class="第1级无序列表">创建一个类，将记录包装起来，并将记录变量的值替换为该类的一个实例。然后在类上定义一个访问函数，用于返回原始的记录。修改封装变量的函数，令其使用这个访问函数。</li>

    <li class="第1级无序列表">测试。</li>

    <li class="第1级无序列表">新建一个函数，让它返回该类的对象，而非那条原始的记录。</li>

    <li class="第1级无序列表">对于该记录的每处使用点，将原先返回记录的函数调用替换为那个返回实例对象的函数调用。使用对象上的访问函数来获取数据的字段，如果该字段的访问函数还不存在，那就创建一个。每次更改之后运行测试。</li>
  </ul>

  <blockquote>
    <p class="zw">如果该记录比较复杂，例如是个嵌套解构，那么先重点关注客户端对数据的更新操作，对于读取操作可以考虑返回一个数据副本或只读的数据代理。</p>
  </blockquote>

  <ul>
    <li class="第1级无序列表">移除类对原始记录的访问函数，那个容易搜索的返回原始数据的函数也要一并删除。</li>

    <li class="第1级无序列表">测试。</li>

    <li class="第1级无序列表">如果记录中的字段本身也是复杂结构，考虑对其再次应用<span style="">封装记录（162）</span>或<span style="">封装集合（170）</span>手法。</li>
  </ul>

  <h3 class="sigil_not_in_toc" id="nav_point_159">范例</h3>

  <p class="zw">首先，我从一个常量开始，该常量在程序中被大量使用。</p>
  <pre class="代码无行号"><code>const organization = {name: "Acme Gooseberries", country: "GB"};</code></pre>

  <p class="zw">这是一个普通的JavaScript对象，程序中很多地方都把它当作记录型结构在使用。以下是对其进行读取和更新的地方：</p>
  <pre class="代码无行号"><code>result += `&lt;h1&gt;${organization.name}&lt;/h1&gt;`;
organization.name = newName;</code></pre>

  <p class="zw">重构的第一步很简单，先施展一下<span style="">封装变量（132）</span>。</p>
  <pre class="代码无行号"><code>function getRawDataOfOrganization() {return organization;}</code></pre>

  <h5 class="sigil_not_in_toc">读取的例子...</h5>
  <pre class="代码无行号"><code>result += `&lt;h1&gt;${getRawDataOfOrganization().name}&lt;/h1&gt;`;</code></pre>

  <h5 class="sigil_not_in_toc">更新的例子...</h5>
  <pre class="代码无行号"><code>getRawDataOfOrganization().name = newName;</code></pre>

  <p class="zw">这里施展的不全是标准的<span style="">封装变量（132）</span>手法，我刻意为设值函数取了一个又丑又长、容易搜索的名字，因为我有意不让它在这次重构中活得太久。</p>

  <p class="zw">封装记录意味着，仅仅替换变量还不够，我还想控制它的使用方式。我可以用类来替换记录，从而达到这一目的。</p>

  <h5 class="sigil_not_in_toc">class Organization...</h5>
  <pre class="代码无行号"><code>class Organization { 
  constructor(data) {
    this._data = data;
  }
}</code></pre>

  <h5 class="sigil_not_in_toc">顶层作用域</h5>
  <pre class="代码无行号"><code>const organization = new Organization({name: "Acme Gooseberries", country: "GB"});

function getRawDataOfOrganization() {return organization._data;}
function getOrganization() {return organization;}</code></pre>

  <p class="zw">创建完对象后，我就能开始寻找该记录的使用点了。所有更新记录的地方，用一个设值函数来替换它。</p>

  <h5 class="sigil_not_in_toc">class Organization...</h5>
  <pre class="代码无行号"><code>set name(aString) {this._data.name = aString;}</code></pre>

  <h5 class="sigil_not_in_toc">客户端...</h5>
  <pre class="代码无行号"><code>getOrganization().name = newName;</code></pre>

  <p class="zw">同样地，我将所有读取记录的地方，用一个取值函数来替代。</p>

  <h5 class="sigil_not_in_toc">class Organization...</h5>
  <pre class="代码无行号"><code>get name() {return this._data.name;}</code></pre>

  <h5 class="sigil_not_in_toc">客户端...</h5>
  <pre class="代码无行号"><code>result += `&lt;h1&gt;${getOrganization().name}&lt;/h1&gt;`;</code></pre>

  <p class="zw">完成引用点的替换后，就可以兑现我之前的死亡威胁，为那个名称丑陋的函数送终了。</p>
  <pre class="代码无行号"><code><del>function getRawDataOfOrganization() {return organization._data;}</del>
function getOrganization() {return organization;}</code></pre>

  <p class="zw">我还倾向于把<code>_data</code>里的字段展开到对象中。</p>
  <pre class="代码无行号"><code>class Organization { 
　constructor(data) {
　　this._name = data.name; 
　　this._country = data.country;
　}
　get name()    {return this._name;}
　set name(aString) {this._name = aString;} 
　get country()    {return this._country;}
　set country(aCountryCode) {this._country = aCountryCode;}
}</code></pre>

  <p class="zw">这样做有一个好处，能够使外界无须再引用原始的数据记录。直接持有原始的记录会破坏封装的完整性。但有时也可能不适合将对象展开到独立的字段里，此时我就会先将<code>_data</code>复制一份，再进行赋值。</p>

  <h3 class="sigil_not_in_toc" id="nav_point_160">范例：封装嵌套记录</h3>

  <p class="zw">上面的例子将记录的浅复制展开到了对象里，但当我处理深层嵌套的数据（比如来自JSON文件的数据）时，又该怎么办呢？此时该重构手法的核心步骤依然适用，记录的更新点需要同样小心处理，但对记录的读取点则有多种处理方案。</p>

  <p class="zw">作为例子，这里有一个嵌套层级更深的数据：它是一组顾客信息的集合，保存在散列映射中，并通过顾客ID进行索引。</p>
  <pre class="代码无行号"><code>"1920": {
　name: "martin", 
　id: "1920",
　usages: { 
　　"2016": {
　　　"1": 50,
　　　"2": 55,
　　　// remaining months of the year
　　}, 
　　"2015": {
　　　"1": 70,
　　　"2": 63,
　　　// remaining months of the year
　　}
　}
},
"38673": {
　name: "neal",
　id: "38673",
　// more customers in a similar form</code></pre>

  <p class="zw">对嵌套数据的更新和读取可以进到更深的层级。</p>

  <h5 class="sigil_not_in_toc">更新的例子...</h5>
  <pre class="代码无行号"><code>customerData[customerID].usages[year][month] = amount;</code></pre>

  <h5 class="sigil_not_in_toc">读取的例子...</h5>
  <pre class="代码无行号"><code>function compareUsage (customerID, laterYear, month) {
　const later = customerData[customerID].usages[laterYear][month]; 
　const earlier = customerData[customerID].usages[laterYear - 1][month]; 
　return {laterAmount: later, change: later - earlier};
}</code></pre>

  <p class="zw">对这样的数据施行封装，第一步仍是<span style="">封装变量（132）</span>。</p>
  <pre class="代码无行号"><code>function getRawDataOfCustomers() {return customerData;} 
function setRawDataOfCustomers(arg) {customerData = arg;}</code></pre>

  <h5 class="sigil_not_in_toc">更新的例子...</h5>
  <pre class="代码无行号"><code>getRawDataOfCustomers()[customerID].usages[year][month] = amount;</code></pre>

  <h5 class="sigil_not_in_toc">读取的例子...</h5>
  <pre class="代码无行号"><code>function compareUsage (customerID, laterYear, month) {
  const later = getRawDataOfCustomers()[customerID].usages[laterYear][month]; 
  const earlier = getRawDataOfCustomers()[customerID].usages[laterYear - 1][month]; 
  return {laterAmount: later, change: later - earlier};
}</code></pre>

  <p class="zw">接下来我要创建一个类来容纳整个数据结构。</p>
  <pre class="代码无行号"><code>class CustomerData { 
  constructor(data) {
    this._data = data;
  }
}</code></pre>

  <h5 class="sigil_not_in_toc">顶层作用域...</h5>
  <pre class="代码无行号"><code>function getCustomerData() {return customerData;}
function getRawDataOfCustomers() {return customerData._data;}
function setRawDataOfCustomers(arg) {customerData = new CustomerData(arg);}</code></pre>

  <p class="zw">最重要的是妥善处理好那些更新操作。因此，当我查看<code>getRawDataOfCustomers</code>的所有调用者时，总是特别关注那些对数据做修改的地方。再提醒你一下，下面是那步更新操作。</p>

  <h5 class="sigil_not_in_toc">更新的例子...</h5>
  <pre class="代码无行号"><code>getRawDataOfCustomers()[customerID].usages[year][month] = amount;</code></pre>

  <p class="zw">“做法”部分说，接下来要通过一个访问函数来返回原始的顾客数据，如果访问函数还不存在就创建一个。现在顾客类还没有设值函数，而且这个更新操作对结构进行了深入查找，因此是时候创建一个设值函数了。我会先用<span style="">提炼函数（106）</span>，将层层深入数据结构的查找操作提炼到函数里。</p>

  <h5 class="sigil_not_in_toc">更新的例子...</h5>
  <pre class="代码无行号"><code>setUsage(customerID, year, month, amount);</code></pre>

  <h5 class="sigil_not_in_toc">顶层作用域...</h5>
  <pre class="代码无行号"><code>function setUsage(customerID, year, month, amount) { 
  getRawDataOfCustomers()[customerID].usages[year][month] = amount;
}</code></pre>

  <p class="zw">然后我再用<span style="">搬移函数（198）</span>将新函数搬移到新的顾客数据类中。</p>

  <h5 class="sigil_not_in_toc">更新的例子...</h5>
  <pre class="代码无行号"><code>getCustomerData().setUsage(customerID, year, month, amount);</code></pre>

  <h5 class="sigil_not_in_toc">class CustomerData...</h5>
  <pre class="代码无行号"><code>setUsage(customerID, year, month, amount) { 
  this._data[customerID].usages[year][month] = amount;
}</code></pre>

  <p class="zw">封装大型的数据结构时，我会更多关注更新操作。凸显更新操作，并将它们集中到一处地方，是此次封装过程最重要的一部分。</p>

  <p class="zw">一通替换过后，我可能认为修改已经告一段落，但如何确认替换是否真正完成了呢？检查的办法有很多，比如可以修改<code>getRawDataOfCustomers</code>函数，让其返回一份数据的深复制的副本。如果测试覆盖足够全面，那么当我真的遗漏了一些更新点时，测试就会报错。</p>

  <h5 class="sigil_not_in_toc">顶层作用域...</h5>
  <pre class="代码无行号"><code>function getCustomerData() {return customerData;}
function getRawDataOfCustomers() {return customerData.rawData;}
function setRawDataOfCustomers(arg) {customerData = new CustomerData(arg);}</code></pre>

  <h5 class="sigil_not_in_toc">class CustomerData...</h5>
  <pre class="代码无行号"><code>get rawData() {
  return _.cloneDeep(this._data);
}</code></pre>

  <p class="zw">我使用了lodash库来辅助生成深复制的副本。</p>

  <p class="zw">另一个方式是，返回一份只读的数据代理。如果客户端代码尝试修改对象的结构，那么该数据代理就会抛出异常。这在有些编程语言中能轻易实现，但用JavaScript实现可就麻烦了，我把它留给读者作为练习好了。或者，我可以复制一份数据，递归冻结副本的每个字段，以此阻止对它的任何修改企图。</p>

  <p class="zw">妥善处理好数据的更新当然价值不凡，但读取操作又怎么处理呢？这有几种选择。</p>

  <p class="zw">第一种选择是与设值函数采用同等待遇，把所有对数据的读取提炼成函数，并将它们搬移到<code>CustomerData</code>类中。</p>

  <h5 class="sigil_not_in_toc">class CustomerData...</h5>
  <pre class="代码无行号"><code>usage(customerID, year, month) {
  return this._data[customerID].usages[year][month];
}</code></pre>

  <h5 class="sigil_not_in_toc">顶层作用域...</h5>
  <pre class="代码无行号"><code>function compareUsage (customerID, laterYear, month) {
  const later = getCustomerData().usage(customerID, laterYear, month); 
  const earlier = getCustomerData().usage(customerID, laterYear - 1, month); 
  return {laterAmount: later, change: later - earlier};
}</code></pre>

  <p class="zw">这种处理方式的美妙之处在于，它为<code>customerData</code>提供了一份清晰的API列表，清楚描绘了该类的全部用途。我只需阅读类的代码，就能知道数据的所有用法。但这样会使代码量剧增，特别是当对象有许多用途时。现代编程语言大多提供直观的语法，以支持从深层的列表和散列[mf-lh]结构中获得数据，因此直接把这样的数据结构给到客户端，也不失为一种选择。</p>

  <p class="zw">如果客户端想拿到一份数据结构，我大可以直接将实际的数据交出去。但这样做的问题在于，我将无从阻止用户直接对数据进行修改，进而使我们封装所有更新操作的良苦用心失去意义。最简单的应对办法是返回原始数据的一份副本，这可以用到我前面写的<code>rawData</code>方法。</p>

  <h5 class="sigil_not_in_toc">class CustomerData...</h5>
  <pre class="代码无行号"><code>get rawData() {
  return _.cloneDeep(this._data);
}</code></pre>

  <h5 class="sigil_not_in_toc">顶层作用域...</h5>
  <pre class="代码无行号"><code>function compareUsage (customerID, laterYear, month) {
  const later = getCustomerData().rawData[customerID].usages[laterYear][month]; 
  const earlier = getCustomerData().rawData[customerID].usages[laterYear - 1][month]; 
  return {laterAmount: later, change: later - earlier};
}</code></pre>

  <p class="zw">简单归简单，这种方案也有缺点。最明显的问题是复制巨大的数据结构时代价颇高，这可能引发性能问题。不过也正如我对性能问题的一贯态度，这样的性能损耗也许是可以接受的——只有测量到可见的影响，我才会真的关心它。这种方案还可能带来困惑，比如客户端可能期望对该数据的修改会同时反映到原数据上。如果采用了只读代理或冻结副本数据的方案，就可以在此时提供一个有意义的错误信息。</p>

  <p class="zw">另一种方案需要更多工作，但能提供更可靠的控制粒度：对每个字段循环应用封装记录。我会把顾客（customer）记录变成一个类，对其用途（usage）字段应用<span style="">封装集合（170），</span>并为它创建一个类。然后我就能通过访问函数来控制其更新点，比如说对用途（usage）对象应用<span style="">将引用对象改为值对象（252）</span>。但处理一个大型的数据结构时，这种方案异常繁复，如果对该数据结构的更新点没那么多，其实大可不必这么做。有时，合理混用取值函数和新对象可能更明智，即使用取值函数来封装数据的深层查找操作，但更新数据时则用对象来包装其结构，而非直接操作未经封装的数据。我在“Refactoring Code to Load a Document”<span style="">[mf-ref-doc]</span>这篇文章中讨论了更多的细节，有兴趣的读者可移步阅读。</p>

  <h2 id="nav_point_161">7.2　封装集合（Encapsulate Collection）</h2>

  <p class="图"><img alt="" src="../Images/image00306.jpeg" style="width: 40%" width="40%"/></p>
  <pre class="代码无行号"><code>class Person {
  get courses() {return this._courses;}
  set courses(aList) {this._courses = aList;}</code></pre>

  <p class="图"><img alt="图像说明文字" src="../Images/image00307.jpeg" style="width: 7%" width="7%"/></p>
  <pre class="代码无行号"><code>class Person {
  get courses() {return this._courses.slice();} 
  addCourse(aCourse) { ... } 
  removeCourse(aCourse) { ... }</code></pre>

  <h3 class="sigil_not_in_toc" id="nav_point_162">动机</h3>

  <p class="zw">我喜欢封装程序中的所有可变数据。这使我很容易看清楚数据被修改的地点和修改方式，这样当我需要更改数据结构时就非常方便。我们通常鼓励封装——使用面向对象技术的开发者对封装尤为重视——但封装集合时人们常常犯一个错误：只对集合变量的访问进行了封装，但依然让取值函数返回集合本身。这使得集合的成员变量可以直接被修改，而封装它的类则全然不知，无法介入。</p>

  <p class="zw">为避免此种情况，我会在类上提供一些修改集合的方法——通常是“添加”和“移除”方法。这样就可使对集合的修改必须经过类，当程序演化变大时，我依然能轻易找出修改点。</p>

  <p class="zw">只要团队拥有良好的习惯，就不会在模块以外修改集合，仅仅提供这些修改方法似乎也就足够。然而，依赖于别人的好习惯是不明智的，一个细小的疏忽就可能带来难以调试的bug。更好的做法是，不要让集合的取值函数返回原始集合，这就避免了客户端的意外修改。</p>

  <p class="zw">一种避免直接修改集合的方法是，永远不直接返回集合的值。这种方法提倡，不要直接使用集合的字段，而是通过定义类上的方法来代替，比如将<code>aCustomer.orders.size</code>替换为<code>aCustomer.numberOfOrders</code>。我不同意这种做法。现代编程语言都提供了丰富的集合类和标准接口，能够组合成很多有价值的用法，比如集合管道（Collection Pipeline）[mf-cp]等。使用特殊的类方法来处理这些场景，会增加许多额外代码，使集合操作容易组合的特性大打折扣。</p>

  <p class="zw">还有一种方法是，以某种形式限制集合的访问权，只允许对集合进行读操作。比如，在Java中可以很容易地返回集合的一个只读代理，这种代理允许用户读取集合，但会阻止所有更改操作——Java的代理会抛出一个异常。有一些库在构造集合时也用了类似的方法，将构造出的集合建立在迭代器或枚举对象的基础上，因为迭代器也不能修改它迭代的集合。</p>

  <p class="zw">也许最常见的做法是，为集合提供一个取值函数，但令其返回一个集合的副本。这样即使有人修改了副本，被封装的集合也不会受到影响。这可能带来一些困惑，特别是对那些已经习惯于通过修改返回值来修改原集合的开发者——但更多的情况下，开发者已经习惯于取值函数返回副本的做法。如果集合很大，这个做法可能带来性能问题，好在多数列表都没有那么大，此时前述的性能优化基本守则依然适用（见2.8节）。</p>

  <p class="zw">使用数据代理和数据复制的另一个区别是，对源数据的修改会反映到代理上，但不会反映到副本上。大多数时候这个区别影响不大，因为通过此种方式访问的列表通常生命周期都不长。</p>

  <p class="zw">采用哪种方法并无定式，最重要的是在同个代码库中做法要保持一致。我建议只用一种方案，这样每个人都能很快习惯它，并在每次调用集合的访问函数时期望相同的行为。</p>

  <h3 class="sigil_not_in_toc" id="nav_point_163">做法</h3>

  <ul>
    <li class="第1级无序列表">如果集合的引用尚未被封装起来，先用<span style="">封装变量（132）</span>封装它。</li>

    <li class="第1级无序列表">在类上添加用于“添加集合元素”和“移除集合元素”的函数。</li>
  </ul>

  <blockquote>
    <p class="zw">如果存在对该集合的设值函数，尽可能先用<span style="">移除设值函数（331）</span>移除它。如果不能移除该设值函数，至少让它返回集合的一份副本。</p>
  </blockquote>

  <ul>
    <li class="第1级无序列表">执行静态检查。</li>

    <li class="第1级无序列表">查找集合的引用点。如果有调用者直接修改集合，令该处调用使用新的添加/移除元素的函数。每次修改后执行测试。</li>

    <li class="第1级无序列表">修改集合的取值函数，使其返回一份只读的数据，可以使用只读代理或数据副本。</li>

    <li class="第1级无序列表">测试。</li>
  </ul>

  <h3 class="sigil_not_in_toc" id="nav_point_164">范例</h3>

  <p class="zw">假设有个人（<code>Person</code>）要去上课。我们用一个简单的<code>Course</code>来表示“课程”。</p>

  <h5 class="sigil_not_in_toc">class Person...</h5>
  <pre class="代码无行号"><code>constructor (name) { 
  this._name = name; 
  this._courses = [];
}
get name() {return this._name;}
get courses() {return this._courses;}
set courses(aList) {this._courses = aList;}</code></pre>

  <h5 class="sigil_not_in_toc">class Course...</h5>
  <pre class="代码无行号"><code>constructor(name, isAdvanced) { 
  this._name = name; 
  this._isAdvanced = isAdvanced;
}
get name() {return this._name;}
get isAdvanced() {return this._isAdvanced;}</code></pre>

  <p class="zw">客户端会使用课程集合来获取课程的相关信息。</p>
  <pre class="代码无行号"><code>numAdvancedCourses = aPerson.courses
  .f ilter(c =&amp;gt; c.isAdvanced)
  .length
;</code></pre>

  <p class="zw">有些开发者可能觉得这个类已经得到了恰当的封装，毕竟，所有的字段都被访问函数保护到了。但我要指出，对课程列表的封装还不完整。诚然，对列表整体的任何更新操作，都能通过设值函数得到控制。</p>

  <h5 class="sigil_not_in_toc">客户端代码...</h5>
  <pre class="代码无行号"><code>const basicCourseNames = readBasicCourseNames(filename);
aPerson.courses = basicCourseNames.map(name =&gt; new Course(name, false));</code></pre>

  <p class="zw">但客户端也可能发现，直接更新课程列表显然更容易。</p>

  <h5 class="sigil_not_in_toc">客户端代码...</h5>
  <pre class="代码无行号"><code>for(const name of readBasicCourseNames(filename)) { 
  aPerson.courses.push(new Course(name, false));
}</code></pre>

  <p class="zw">这就破坏了封装性，因为以此种方式更新列表<code>Person</code>类根本无从得知。这里仅仅封装了字段引用，而未真正封装字段的内容。</p>

  <p class="zw">现在我来对类实施真正恰当的封装，首先要为类添加两个方法，为客户端提供“添加课程”和“移除课程”的接口。</p>

  <h5 class="sigil_not_in_toc">class Person...</h5>
  <pre class="代码无行号"><code>addCourse(aCourse) { 
  this._courses.push(aCourse);
}
removeCourse(aCourse, fnIfAbsent = () =&gt; {throw new RangeError();}) { 
  const index = this._courses.indexOf(aCourse);
  if (index === -1) fnIfAbsent();
  else this._courses.splice(index, 1);
}</code></pre>

  <p class="zw">对于移除操作，我得考虑一下，如果客户端要求移除一个不存在的集合元素怎么办。我可以耸耸肩装作没看见，也可以抛出错误。这里我默认让它抛出错误，但留给客户端一个自己处理的机会。</p>

  <p class="zw">然后我就可以让直接修改集合值的地方改用新的方法了。</p>

  <h5 class="sigil_not_in_toc">客户端代码...</h5>
  <pre class="代码无行号"><code>for(const name of readBasicCourseNames(filename)) { 
  aPerson.addCourse(new Course(name, false));
}</code></pre>

  <p class="zw">有了单独的添加和移除方法，通常<code>setCourse</code>设值函数就没必要存在了。若果真如此，我就会使用<span style="">移除设值函数（331）</span>移除它。如果出于其他原因，必须提供一个设值方法作为API，我至少要确保用一份副本给字段赋值，不去修改通过参数传入的集合。</p>

  <h5 class="sigil_not_in_toc">class Person...</h5>
  <pre class="代码无行号"><code>set courses(aList) {this._courses = aList.slice();}</code></pre>

  <p class="zw">这套设施让客户端能够使用正确的修改方法，同时我还希望能确保所有修改都通过这些方法进行。为达此目的，我会让取值函数返回一份副本。</p>

  <h5 class="sigil_not_in_toc">class Person...</h5>
  <pre class="代码无行号"><code>get courses() {return this._courses.slice();}</code></pre>

  <p class="zw">总的来讲，我觉得对集合保持适度的审慎是有益的，我宁愿多复制一份数据，也不愿去调试因意外修改集合招致的错误。修改操作并不总是显而易见的，比如，在JavaScript中原生的数组排序函数<code>sort()</code>就会修改原数组，而在其他语言中默认都是为更改集合的操作返回一份副本。任何负责管理集合的类都应该总是返回数据副本，但我还养成了一个习惯，只要我做的事看起来可能改变集合，我也会返回一个副本。</p>

  <h2 id="nav_point_165">7.3　以对象取代基本类型（Replace Primitive with Object）</h2>

  <p class="zw">曾用名：<span style="">以对象取代数据值（Replace Data Value with Object）</span></p>

  <p class="zw">曾用名：<span style="">以类取代类型码（Replace Type Code with Class）</span></p>

  <p class="图"><img alt="" src="../Images/image00308.jpeg" style="width: 40%" width="40%"/></p>
  <pre class="代码无行号"><code>orders.filter(o =&gt; "high" === o.priority
               || "rush" === o.priority);</code></pre>

  <p class="图"><img alt="图像说明文字" src="../Images/image00309.jpeg" style="width: 7%" width="7%"/></p>
  <pre class="代码无行号"><code>orders.filter(o =&gt; o.priority.higherThan(new Priority("normal")))</code></pre>

  <h3 class="sigil_not_in_toc" id="nav_point_166">动机</h3>

  <p class="zw">开发初期，你往往决定以简单的数据项表示简单的情况，比如使用数字或字符串等。但随着开发的进行，你可能会发现，这些简单数据项不再那么简单了。比如说，一开始你可能会用一个字符串来表示“电话号码”的概念，但是随后它又需要“格式化”“抽取区号”之类的特殊行为。这类逻辑很快便会占领代码库，制造出许多重复代码，增加使用时的成本。</p>

  <p class="zw">一旦我发现对某个数据的操作不仅仅局限于打印时，我就会为它创建一个新类。一开始这个类也许只是简单包装一下简单类型的数据，不过只要类有了，日后添加的业务逻辑就有地可去了。这些小小的封装值开始可能价值甚微，但只要悉心照料，它们很快便能成长为有用的工具。创建新类无须太大的工作量，但我发现它们往往对代码库有深远的影响。实际上，许多经验丰富的开发者认为，这是他们的工具箱里最实用的重构手法之一——尽管其价值常为新手程序员所低估。</p>

  <h3 class="sigil_not_in_toc" id="nav_point_167">做法</h3>

  <ul>
    <li class="第1级无序列表">如果变量尚未被封装起来，先使用<span style="">封装变量（132）</span>封装它。</li>

    <li class="第1级无序列表">为这个数据值创建一个简单的类。类的构造函数应该保存这个数据值，并为它提供一个取值函数。</li>

    <li class="第1级无序列表">执行静态检查。</li>

    <li class="第1级无序列表">修改第一步得到的设值函数，令其创建一个新类的对象并将其存入字段，如果有必要的话，同时修改字段的类型声明。</li>

    <li class="第1级无序列表">修改取值函数，令其调用新类的取值函数，并返回结果。</li>

    <li class="第1级无序列表">测试。</li>

    <li class="第1级无序列表">考虑对第一步得到的访问函数使用<span style="">函数改名（124）</span>，以便更好反映其用途。</li>

    <li class="第1级无序列表">考虑应用<span style="">将引用对象改为值对象（252）</span>或<span style="">将值对象改为引用对象（256）</span>，明确指出新对象的角色是值对象还是引用对象。</li>
  </ul>

  <h3 class="sigil_not_in_toc" id="nav_point_168">范例</h3>

  <p class="zw">我将从一个简单的订单（<code>Order</code>）类开始。该类从一个简单的记录结构里读取所需的数据，这其中有一个订单优先级（<code>priority</code>）字段，它是以字符串的形式被读入的。</p>

  <h5 class="sigil_not_in_toc">class Order...</h5>
  <pre class="代码无行号"><code>constructor(data) { 
  this.priority = data.priority;
  // more initialization</code></pre>

  <p class="zw">客户端代码有些地方是这么用它的：</p>

  <h5 class="sigil_not_in_toc">客户端...</h5>
  <pre class="代码无行号"><code>highPriorityCount = orders.filter(o =&gt; "high" === o.priority
                                   || "rush" === o.priority)
                          .length;</code></pre>

  <p class="zw">无论何时，当我与一个数据值打交道时，第一件事一定是对它使用<span style="">封装变量（132）</span>。</p>

  <h5 class="sigil_not_in_toc">class Order...</h5>
  <pre class="代码无行号"><code>get priority()        {return this._priority;} 
set priority(aString) {this._priority = aString;}</code></pre>

  <p class="zw">现在构造函数中第一行初始化代码就会使用我刚刚创建的设值函数了。</p>

  <p class="zw">这使它成了一个自封装的字段，因此我暂可放任原来的引用点不理，先对字段进行处理。</p>

  <p class="zw">接下来我为优先级字段创建一个简单的值类（value class）。该类应该有一个构造函数接收值字段，并提供一个返回字符串的转换函数。</p>
  <pre class="代码无行号"><code>class Priority {
  constructor(value) {this._value = value;} 
  toString() {return this._value;}
}</code></pre>

  <p class="zw">这里的转换函数我更倾向于使用<code>toString</code>而不用取值函数（<code>value</code>）。对类的客户端而言，一个返回字符串描述的API应该更能传达“发生了数据转换”的信息，而使用取值函数取用一个字段就缺乏这方面的感觉。</p>

  <p class="zw">然后我要修改访问函数，使其用上新创建的类。</p>

  <h5 class="sigil_not_in_toc">class Order...</h5>
  <pre class="代码无行号"><code>get priority()        {return this._priority.toString();}
set priority(aString) {this._priority = new Priority(aString);}</code></pre>

  <p class="zw">提炼出<code>Priority</code>类后，我发觉现在<code>Order</code>类上的取值函数命名有点儿误导人了。它确实还是返回了优先级信息，但却是一个字符串描述，而不是一个<code>Priority</code>对象。于是我立即对它应用了<span style="">函数改名（124）</span>。</p>

  <h5 class="sigil_not_in_toc">class Order...</h5>
  <pre class="代码无行号"><code>get priorityString() {return this._priority.toString();}
set priority(aString) {this._priority = new Priority(aString);}</code></pre>

  <h5 class="sigil_not_in_toc">客户端...</h5>
  <pre class="代码无行号"><code>highPriorityCount = orders.filter(o =&gt; "high" === o.priorityString
                                   || "rush" === o.priorityString)
                          .length;</code></pre>

  <p class="zw">这里设值函数的名字倒没有使我不满，因为函数的参数能够清晰地表达其意图。</p>

  <p class="zw">到此为止，正式的重构手法就结束了。不过当我进一步查看优先级字段的客户端时，我在想让它们直接使用<code>Priority</code>对象是否会更好。于是，我着手在订单类上添加一个取值函数，让它直接返回新建的<code>Priority</code>对象。</p>

  <h5 class="sigil_not_in_toc">class Order...</h5>
  <pre class="代码无行号"><code>get priority()        {return this._priority;}
get priorityString()  {return this._priority.toString();}
set priority(aString) {this._priority = new Priority(aString);}</code></pre>

  <h5 class="sigil_not_in_toc">客户端...</h5>
  <pre class="代码无行号"><code>highPriorityCount = orders.filter(o =&gt; "high" === o.priority.toString()
                                   || "rush" === o.priority.toString())
                          .length;</code></pre>

  <p class="zw">随着<code>Priority</code>对象在别处也有了用处，我开始支持让<code>Order</code>类的客户端拿着<code>Priority</code>实例来调用设值函数，这可以通过调整<code>Priority</code>类的构造函数实现。</p>

  <h5 class="sigil_not_in_toc">class Priority...</h5>
  <pre class="代码无行号"><code>constructor(value) {
  if (value instanceof Priority) return value;
  this._value = value;
}</code></pre>

  <p class="zw">这样做的意义在于，现在新的<code>Priority</code>类可以容纳更多业务行为——无论是新的业务代码，还是从别处搬移过来的。这里有些例子，它会校验优先级的传入值，支持一些比较逻辑。</p>

  <h5 class="sigil_not_in_toc">class Priority...</h5>
  <pre class="代码无行号"><code>constructor(value) {
  if (value instanceof Priority) return value; 
  if (Priority.legalValues().includes(value))
    this._value = value; 
  else
    throw new Error(`&lt;${value}&gt; is invalid for Priority`);
}
toString() {return this._value;}
get _index() {return Priority.legalValues().findIndex(s =&gt; s === this._value);} 
static legalValues() {return ['low', 'normal', 'high', 'rush'];}

equals(other) {return this._index === other._index;} 
higherThan(other) {return this._index &gt; other._index;} 
lowerThan(other) {return this._index &lt; other._index;}</code></pre>

  <p class="zw">修改的过程中，我发觉它实际上已经担负起值对象（value object）的角色，因此我又为它添加了一个<code>equals</code>方法，并确保它的值不可修改。</p>

  <p class="zw">加上这些行为后，我可以让客户端代码读起来含义更清晰。</p>

  <h5 class="sigil_not_in_toc">客户端...</h5>
  <pre class="代码无行号"><code>highPriorityCount = orders.filter(o =&gt; o.priority.higherThan(new Priority("normal")))
                          .length;</code></pre>

  <h2 id="nav_point_169">7.4　以查询取代临时变量（Replace Temp with Query）</h2>

  <p class="图"><img alt="" src="../Images/image00310.jpeg" style="width: 40%" width="40%"/></p>
  <pre class="代码无行号"><code>const basePrice = this._quantity * this._itemPrice; 
if (basePrice &gt; 1000)
  return basePrice * 0.95; 
else
  return basePrice * 0.98;</code></pre>

  <p class="图"><img alt="图像说明文字" src="../Images/image00311.jpeg" style="width: 7%" width="7%"/></p>
  <pre class="代码无行号"><code>get basePrice() {this._quantity * this._itemPrice;}

...

if (this.basePrice &gt; 1000) 
  return this.basePrice * 0.95;
else
  return this.basePrice * 0.98;</code></pre>

  <h3 class="sigil_not_in_toc" id="nav_point_170">动机</h3>

  <p class="zw">临时变量的一个作用是保存某段代码的返回值，以便在函数的后面部分使用它。临时变量允许我引用之前的值，既能解释它的含义，还能避免对代码进行重复计算。但尽管使用变量很方便，很多时候还是值得更进一步，将它们抽取成函数。</p>

  <p class="zw">如果我正在分解一个冗长的函数，那么将变量抽取到函数里能使函数的分解过程更简单，因为我就不再需要将变量作为参数传递给提炼出来的小函数。将变量的计算逻辑放到函数中，也有助于在提炼得到的函数与原函数之间设立清晰的边界，这能帮我发现并避免难缠的依赖及副作用。</p>

  <p class="zw">改用函数还让我避免了在多个函数中重复编写计算逻辑。每当我在不同的地方看见同一段变量的计算逻辑，我就会想方设法将它们挪到同一个函数里。</p>

  <p class="zw">这项重构手法在类中施展效果最好，因为类为待提炼函数提供了一个共同的上下文。如果不是在类中，我很可能会在顶层函数中拥有过多参数，这将冲淡提炼函数所能带来的诸多好处。使用嵌套的小函数可以避免这个问题，但又限制了我在相关函数间分享逻辑的能力。</p>

  <p class="zw"><span style="">以查询取代临时变量（178）</span>手法只适用于处理某些类型的临时变量：那些只被计算一次且之后不再被修改的变量。最简单的情况是，这个临时变量只被赋值一次，但在更复杂的代码片段里，变量也可能被多次赋值——此时应该将这些计算代码一并提炼到查询函数中。并且，待提炼的逻辑多次计算同样的变量时，应该能得到相同的结果。因此，对于那些做快照用途的临时变量（从变量名往往可见端倪，比如<code>oldAddress</code>这样的名字），就不能使用本手法。</p>

  <h3 class="sigil_not_in_toc" id="nav_point_171">做法</h3>

  <ul>
    <li class="第1级无序列表">检查变量在使用前是否已经完全计算完毕，检查计算它的那段代码是否每次都能得到一样的值。</li>

    <li class="第1级无序列表">如果变量目前不是只读的，但是可以改造成只读变量，那就先改造它。</li>

    <li class="第1级无序列表">测试。</li>

    <li class="第1级无序列表">将为变量赋值的代码段提炼成函数。</li>
  </ul>

  <blockquote>
    <p class="zw">如果变量和函数不能使用同样的名字，那么先为函数取个临时的名字。</p>

    <p class="zw">确保待提炼函数没有副作用。若有，先应用<span style="">将查询函数和修改函数分离（306）</span>手法隔离副作用。</p>
  </blockquote>

  <ul>
    <li class="第1级无序列表">测试。</li>

    <li class="第1级无序列表">应用<span style="">内联变量（123）</span>手法移除临时变量。</li>
  </ul>

  <h3 class="sigil_not_in_toc" id="nav_point_172">范例</h3>

  <p class="zw">这里有一个简单的订单类。</p>

  <h5 class="sigil_not_in_toc">class Order...</h5>
  <pre class="代码无行号"><code>  constructor(quantity, item) { 
    this._quantity = quantity; 
    this._item = item;
  }

  get price() {
    var basePrice = this._quantity * this._item.price; 
    var discountFactor = 0.98;
    if (basePrice &gt; 1000) discountFactor -= 0.03; 
    return basePrice * discountFactor;
  }
}</code></pre>

  <p class="zw">我希望把<code>basePrice</code>和<code>discountFactor</code>两个临时变量变成函数。</p>

  <p class="zw">先从<code>basePrice</code>开始，我先把它声明成<code>const</code>并运行测试。这可以很好地防止我遗漏了对变量的其他赋值点——对于这么个小函数是不太可能的，但当我处理更大的函数时就不一定了。</p>

  <h5 class="sigil_not_in_toc">class Order...</h5>
  <pre class="代码无行号"><code>　constructor(quantity, item) { 
　　this._quantity = quantity; 
　　this._item = item;
　}

　get price() {
　　const basePrice = this._quantity * this._item.price; 
　　var discountFactor = 0.98;
　　if (basePrice &gt; 1000) discountFactor -= 0.03; 
　　return basePrice * discountFactor;
　}
}</code></pre>

  <p class="zw">然后我把赋值操作的右边提炼成一个取值函数。</p>

  <h5 class="sigil_not_in_toc">class Order...</h5>
  <pre class="代码无行号"><code>get price() {
　const basePrice = this.basePrice;
　var discountFactor = 0.98;
　if (basePrice &gt; 1000) discountFactor -= 0.03; 
　return basePrice * discountFactor;
}

　get basePrice() {
　　return this._quantity * this._item.price;
　}</code></pre>

  <p class="zw">测试，然后应用<span style="">内联变量（123）</span>。</p>

  <h5 class="sigil_not_in_toc">class Order...</h5>
  <pre class="代码无行号"><code>get price() {
　<del>const basePrice = this.basePrice;</del>
　var discountFactor = 0.98;
　if (this.basePrice &gt; 1000) discountFactor -= 0.03; 
　return this.basePrice * discountFactor;
}</code></pre>

  <p class="zw">接下来我对<code>discountFactor</code>重复同样的步骤，先是应用<span style="">提炼函数（106）</span>。</p>

  <h5 class="sigil_not_in_toc">class Order...</h5>
  <pre class="代码无行号"><code>get price() {
　const discountFactor = this.discountFactor;
　return this.basePrice * discountFactor;
}

　get discountFactor() {
　　var discountFactor = 0.98;
　　if (this.basePrice &gt; 1000) discountFactor -= 0.03; 
　　return discountFactor;
}</code></pre>

  <p class="zw">这里我需要将对<code>discountFactor</code>的两处赋值一起搬移到新提炼的函数中，之后就可以将原变量一起声明为<code>const</code>。</p>

  <p class="zw">然后，内联变量：</p>
  <pre class="代码无行号"><code>get price() {
  return this.basePrice * this.discountFactor;
}</code></pre>

  <h2 id="nav_point_173">7.5　提炼类（Extract Class）</h2>

  <p class="zw">反向重构：<span style="">内联类（186）</span></p>

  <p class="图"><img alt="" src="../Images/image00312.jpeg" style="width: 40%" width="40%"/></p>
  <pre class="代码无行号"><code>class Person {
　get officeAreaCode() {return this._officeAreaCode;} 
　get officeNumber()   {return this._officeNumber;}</code></pre>

  <p class="图"><img alt="图像说明文字" src="../Images/image00313.jpeg" style="width: 7%" width="7%"/></p>
  <pre class="代码无行号"><code>class Person {
　get officeAreaCode() {return this._telephoneNumber.areaCode;} 
　get officeNumber()   {return this._telephoneNumber.number;}
}
class TelephoneNumber {
　get areaCode() {return this._areaCode;} 
　get number()   {return this._number;}
}</code></pre>

  <h3 class="sigil_not_in_toc" id="nav_point_174">动机</h3>

  <p class="zw">你也许听过类似这样的建议：一个类应该是一个清晰的抽象，只处理一些明确的责任，等等。但是在实际工作中，类会不断成长扩展。你会在这儿加入一些功能，在那儿加入一些数据。给某个类添加一项新责任时，你会觉得不值得为这项责任分离出一个独立的类。于是，随着责任不断增加，这个类会变得过分复杂。很快，你的类就会变成一团乱麻。</p>

  <p class="zw">设想你有一个维护大量函数和数据的类。这样的类往往因为太大而不易理解。此时你需要考虑哪些部分可以分离出去，并将它们分离到一个独立的类中。如果某些数据和某些函数总是一起出现，某些数据经常同时变化甚至彼此相依，这就表示你应该将它们分离出去。一个有用的测试就是问你自己，如果你搬移了某些字段和函数，会发生什么事？其他字段和函数是否因此变得无意义？</p>

  <p class="zw">另一个往往在开发后期出现的信号是类的子类化方式。如果你发现子类化只影响类的部分特性，或如果你发现某些特性需要以一种方式来子类化，某些特性则需要以另一种方式子类化，这就意味着你需要分解原来的类。</p>

  <h3 class="sigil_not_in_toc" id="nav_point_175">做法</h3>

  <ul>
    <li class="第1级无序列表">决定如何分解类所负的责任。</li>

    <li class="第1级无序列表">创建一个新的类，用以表现从旧类中分离出来的责任。</li>
  </ul>

  <blockquote>
    <p class="zw">如果旧类剩下的责任与旧类的名称不符，为旧类改名。</p>
  </blockquote>

  <ul>
    <li class="第1级无序列表">构造旧类时创建一个新类的实例，建立“从旧类访问新类”的连接关系。</li>

    <li class="第1级无序列表">对于你想搬移的每一个字段，运用<span style="">搬移字段（207）</span>搬移之。每次更改后运行测试。</li>

    <li class="第1级无序列表">使用<span style="">搬移函数（198）</span>将必要函数搬移到新类。先搬移较低层函数（也就是“被其他函数调用”多于“调用其他函数”者）。每次更改后运行测试。</li>

    <li class="第1级无序列表">检查两个类的接口，去掉不再需要的函数，必要时为函数重新取一个适合新环境的名字。</li>

    <li class="第1级无序列表">决定是否公开新的类。如果确实需要，考虑对新类应用<span style="">将引用对象改为值对象（252）</span>使其成为一个值对象。</li>
  </ul>

  <h3 class="sigil_not_in_toc" id="nav_point_176">范例</h3>

  <p class="zw">我们从一个简单的<code>Person</code>类开始。</p>

  <h5 class="sigil_not_in_toc">class Person...</h5>
  <pre class="代码无行号"><code>get name() 　 {return this._name;} 
set name(arg) {this._name = arg;}
get telephoneNumber() {return `(${this.officeAreaCode}) ${this.officeNumber}`;} 
get officeAreaCode() 　　{return this._officeAreaCode;}
set officeAreaCode(arg)　{this._officeAreaCode = arg;} 
get officeNumber() {return this._officeNumber;}
set officeNumber(arg) {this._officeNumber = arg;}</code></pre>

  <p class="zw">这里，我可以将与电话号码相关的行为分离到一个独立的类中。首先，我要定义一个空的<code>TelephoneNumber</code>类来表示“电话号码”这个概念：</p>
  <pre class="代码无行号"><code>class TelephoneNumber {
}</code></pre>

  <p class="zw">易如反掌！接着，我要在构造<code>Person</code>类时创建<code>TelephoneNumber</code>类的一个实例。</p>

  <h5 class="sigil_not_in_toc">class Person...</h5>
  <pre class="代码无行号"><code>constructor() {
  this._telephoneNumber = new TelephoneNumber();
}</code></pre>

  <h5 class="sigil_not_in_toc">class TelephoneNumber...</h5>
  <pre class="代码无行号"><code>get officeAreaCode()    {return this._officeAreaCode;} 
set officeAreaCode(arg) {this._officeAreaCode = arg;}</code></pre>

  <p class="zw">现在，我运用<span style="">搬移字段（207）</span>搬移一个字段。</p>

  <h5 class="sigil_not_in_toc">class Person...</h5>
  <pre class="代码无行号"><code>get officeAreaCode()    {return this._telephoneNumber.officeAreaCode;} 
set officeAreaCode(arg) {this._telephoneNumber.officeAreaCode = arg;}</code></pre>

  <p class="zw">再次运行测试，然后我对下一个字段进行同样处理。</p>

  <h5 class="sigil_not_in_toc">class TelephoneNumber...</h5>
  <pre class="代码无行号"><code>get officeNumber() {return this._officeNumber;} 
set officeNumber(arg) {this._officeNumber = arg;}</code></pre>

  <h5 class="sigil_not_in_toc">class Person...</h5>
  <pre class="代码无行号"><code>get officeNumber() {return this._telephoneNumber.officeNumber;} 
set officeNumber(arg) {this._telephoneNumber.officeNumber = arg;}</code></pre>

  <p class="zw">再次测试，然后再搬移对电话号码的取值函数。</p>

  <h5 class="sigil_not_in_toc">class TelephoneNumber...</h5>
  <pre class="代码无行号"><code>get telephoneNumber() {return `(${this.officeAreaCode}) ${this.officeNumber}`;}</code></pre>

  <h5 class="sigil_not_in_toc">class Person...</h5>
  <pre class="代码无行号"><code>get telephoneNumber() {return this._telephoneNumber.telephoneNumber;}</code></pre>

  <p class="zw">现在我需要做些清理工作。“电话号码”显然不该拥有“办公室”（office）的概念，因此我得重命名一下变量。</p>

  <h5 class="sigil_not_in_toc">class TelephoneNumber...</h5>
  <pre class="代码无行号"><code>get areaCode()    {return this._areaCode;} 
set areaCode(arg) {this._areaCode = arg;}

get number()    {return this._number;} 
set number(arg) {this._number = arg;}</code></pre>

  <h5 class="sigil_not_in_toc">class Person...</h5>
  <pre class="代码无行号"><code>get officeAreaCode()    {return this._telephoneNumber.areaCode;} 
set officeAreaCode(arg) {this._telephoneNumber.areaCode = arg;} 
get officeNumber()    {return this._telephoneNumber.number;} 
set officeNumber(arg) {this._telephoneNumber.number = arg;}</code></pre>

  <p class="zw"><code>TelephoneNumber</code>类上有一个对自己（telephone number）的取值函数也没什么道理，因此我又对它应用<span style="">函数改名（124）</span>。</p>

  <h5 class="sigil_not_in_toc">class TelephoneNumber...</h5>
  <pre class="代码无行号"><code>toString() {return `(${this.areaCode}) ${this.number}`;}</code></pre>

  <h5 class="sigil_not_in_toc">class Person...</h5>
  <pre class="代码无行号"><code>get telephoneNumber() {return this._telephoneNumber.toString();}</code></pre>

  <p class="zw">“电话号码”对象一般还具有复用价值，因此我考虑将新提炼的类暴露给更多的客户端。需要访问<code>TelephoneNumber</code>对象时，只须把<code>Person</code>类中那些<code>office</code>开头的访问函数搬移过来并略作修改即可。但这样<code>TelephoneNumber</code>就更像一个值对象（Value Object）[mf-vo]了，因此我会先对它使用<span style="">将引用对象改为值对象（252）</span>（那个重构手法所用的范例，正是基于本章电话号码例子的延续）。</p>

  <h2 id="nav_point_177">7.6　内联类（Inline Class）</h2>

  <p class="zw">反向重构：<span style="">提炼类（182）</span></p>

  <p class="图"><img alt="" src="../Images/image00314.jpeg" style="width: 40%" width="40%"/></p>
  <pre class="代码无行号"><code>class Person {
　get officeAreaCode() {return this._telephoneNumber.areaCode;} 
　get officeNumber() 　{return this._telephoneNumber.number;}
}
class TelephoneNumber {
　get areaCode() {return this._areaCode;} 
　get number() {return this._number;}
}</code></pre>

  <p class="图"><img alt="图像说明文字" src="../Images/image00315.jpeg" style="width: 7%" width="7%"/></p>
  <pre class="代码无行号"><code>class Person {
　get officeAreaCode() {return this._officeAreaCode;} 
　get officeNumber()　 {return this._officeNumber;}</code></pre>

  <h3 class="sigil_not_in_toc" id="nav_point_178">动机</h3>

  <p class="zw"><span style="">内联类</span>正好与<span style="">提炼类（182）</span>相反。如果一个类不再承担足够责任，不再有单独存在的理由（这通常是因为此前的重构动作移走了这个类的责任），我就会挑选这一“萎缩类”的最频繁用户（也是一个类），以本手法将“萎缩类”塞进另一个类中。</p>

  <p class="zw">应用这个手法的另一个场景是，我手头有两个类，想重新安排它们肩负的职责，并让它们产生关联。这时我发现先用本手法将它们内联成一个类再用<span style="">提炼类（182）</span>去分离其职责会更加简单。这是重新组织代码时常用的做法：有时把相关元素一口气搬移到位更简单，但有时先用内联手法合并各自的上下文，再使用提炼手法再次分离它们会更合适。</p>

  <h3 class="sigil_not_in_toc" id="nav_point_179">做法</h3>

  <ul>
    <li class="第1级无序列表">对于待内联类（源类）中的所有<code>public</code>函数，在目标类上创建一个对应的函数，新创建的所有函数应该直接委托至源类。</li>

    <li class="第1级无序列表">修改源类<code>public</code>方法的所有引用点，令它们调用目标类对应的委托方法。每次更改后运行测试。</li>

    <li class="第1级无序列表">将源类中的函数与数据全部搬移到目标类，每次修改之后进行测试，直到源类变成空壳为止。</li>

    <li class="第1级无序列表">删除源类，为它举行一个简单的“丧礼”</li>
  </ul>

  <h3 class="sigil_not_in_toc" id="nav_point_180">范例</h3>

  <p class="zw">下面这个类存储了一次物流运输（shipment）的若干跟踪信息（tracking information）。</p>
  <pre class="代码无行号"><code>class TrackingInformation {
　get shippingCompany()    {return this._shippingCompany;} 
　set shippingCompany(arg) {this._shippingCompany = arg;} 
　get trackingNumber()    {return this._trackingNumber;} 
　set trackingNumber(arg) {this._trackingNumber = arg;} 
　get display()            {
　　return `${this.shippingCompany}: ${this.trackingNumber}`;
　}
}</code></pre>

  <p class="zw">它作为<code>Shipment</code>（物流）类的一部分被使用。</p>

  <h5 class="sigil_not_in_toc">class Shipment...</h5>
  <pre class="代码无行号"><code>get trackingInfo() {
　return this._trackingInformation.display;
}
get trackingInformation() {return this._trackingInformation;} 
set trackingInformation(aTrackingInformation) {
　this._trackingInformation = aTrackingInformation;
}</code></pre>

  <p class="zw"><code>TrackingInformation</code>类过去可能有很多光荣职责，但现在我觉得它已不再能肩负起它的责任，因此我希望将它内联到<code>Shipment</code>类里。</p>

  <p class="zw">首先，我要寻找<code>TrackingInformation</code>类的方法有哪些调用点。</p>

  <h5 class="sigil_not_in_toc">调用方...</h5>
  <pre class="代码无行号"><code>aShipment.trackingInformation.shippingCompany = request.vendor;</code></pre>

  <p class="zw">我将开始将源类的类似函数全都搬移到<code>Shipment</code>里去，但我的做法与做<span style="">搬移函数（198）</span>时略微有些不同。这里，我先在<code>Shipment</code>类里创建一个委托方法，并调整客户端代码，使其调用这个委托方法。</p>

  <h5 class="sigil_not_in_toc">class Shipment...</h5>
  <pre class="代码无行号"><code>  set shippingCompany(arg) {this._trackingInformation.shippingCompany = arg;}</code></pre>

  <h5 class="sigil_not_in_toc">调用方...</h5>
  <pre class="代码无行号"><code>aShipment<del>.trackingInformation.</del>shippingCompany = request.vendor;</code></pre>

  <p class="zw">对于<code>TrackingInformation</code>类中所有为客户端调用的方法，我将施以相同的手法。这之后，我就可以将源类中的所有东西都搬移到<code>Shipment</code>类中去。</p>

  <p class="zw">我先对<code>display</code>方法应用<span style="">内联函数（115）</span>手法。</p>

  <h5 class="sigil_not_in_toc">class Shipment...</h5>
  <pre class="代码无行号"><code>get trackingInfo() {
  return `${this.shippingCompany}: ${this.trackingNumber}`;
}</code></pre>

  <p class="zw">再继续搬移“收货公司”（shipping company）字段。</p>
  <pre class="代码无行号"><code>get shippingCompany() {return this<del>._trackingInformation.</del>_shippingCompany;} 
set shippingCompany(arg) {this.<del>_trackingInformation.</del>_shippingCompany = arg;}</code></pre>

  <p class="zw">我并未遵循<span style="">搬移字段（207）</span>的全部步骤，因为此处我只是改由目标类<code>Shipment</code>来引用<code>shippingCompany</code>，那些从源类搬移引用至目标类的步骤在此并不需要。</p>

  <p class="zw">我会继续相同的手法，直到所有搬迁工作完成为止。那时，我就可以删除<code>TrackingInformation</code>类了。</p>

  <h5 class="sigil_not_in_toc">class Shipment...</h5>
  <pre class="代码无行号"><code>get trackingInfo() {
　return `${this.shippingCompany}: ${this.trackingNumber}`;
}
get shippingCompany()    {return this._shippingCompany;} 
set shippingCompany(arg) {this._shippingCompany = arg;} 
get trackingNumber()    {return this._trackingNumber;} 
set trackingNumber(arg) {this._trackingNumber = arg;}</code></pre>

  <h2 id="nav_point_181">7.7　隐藏委托关系（Hide Delegate）</h2>

  <p class="zw">反向重构：<span style="">移除中间人（192）</span></p>

  <p class="图"><img alt="" src="../Images/image00316.jpeg" style="width: 40%" width="40%"/></p>
  <pre class="代码无行号"><code>manager = aPerson.department.manager;</code></pre>

  <p class="图"><img alt="图像说明文字" src="../Images/image00317.jpeg" style="width: 7%" width="7%"/></p>
  <pre class="代码无行号"><code>manager = aPerson.manager; 

class Person {
  get manager() {return this.department.manager;}</code></pre>

  <h3 class="sigil_not_in_toc" id="nav_point_182">动机</h3>

  <p class="zw">一个好的模块化的设计，“封装”即使不是其最关键特征，也是最关键特征之一。“封装”意味着每个模块都应该尽可能少了解系统的其他部分。如此一来，一旦发生变化，需要了解这一变化的模块就会比较少——这会使变化比较容易进行。</p>

  <p class="zw">当我们初学面向对象技术时就被教导，封装意味着应该隐藏自己的字段。随着经验日渐丰富，你会发现，有更多可以（而且值得）封装的东西。</p>

  <p class="zw">如果某些客户端先通过服务对象的字段得到另一个对象（受托类），然后调用后者的函数，那么客户就必须知晓这一层委托关系。万一受托类修改了接口，变化会波及通过服务对象使用它的所有客户端。我可以在服务对象上放置一个简单的委托函数，将委托关系隐藏起来，从而去除这种依赖。这么一来，即使将来委托关系发生变化，变化也只会影响服务对象，而不会直接波及所有客户端。</p>

  <p class="图"><img alt="" src="../Images/image00318.jpeg" style="width: 80%" width="80%"/></p>

  <h3 class="sigil_not_in_toc" id="nav_point_183">做法</h3>

  <ul>
    <li class="第1级无序列表">对于每个委托关系中的函数，在服务对象端建立一个简单的委托函数。</li>

    <li class="第1级无序列表">调整客户端，令它只调用服务对象提供的函数。每次调整后运行测试。</li>

    <li class="第1级无序列表">如果将来不再有任何客户端需要取用<code>Delegate</code>（受托类），便可移除服务对象中的相关访问函数。</li>

    <li class="第1级无序列表">测试。</li>
  </ul>

  <h3 class="sigil_not_in_toc" id="nav_point_184">范例</h3>

  <p class="zw">本例从两个类开始，代表“人”的<code>Person</code>和代表“部门”的<code>Department</code>。</p>

  <h5 class="sigil_not_in_toc">class Person...</h5>
  <pre class="代码无行号"><code>constructor(name) { 
　this._name = name;
}
get name() {return this._name;}
get department()    {return this._department;} 
set department(arg) {this._department = arg;}</code></pre>

  <h5 class="sigil_not_in_toc">class Department...</h5>
  <pre class="代码无行号"><code>get chargeCode() {return this._chargeCode;} 
set chargeCode(arg) {this._chargeCode = arg;} 
get manager() {return this._manager;}
set manager(arg) {this._manager = arg;}</code></pre>

  <p class="zw">有些客户端希望知道某人的经理是谁，为此，它必须先取得<code>Department</code>对象。</p>

  <h5 class="sigil_not_in_toc">客户端代码...</h5>
  <pre class="代码无行号"><code>manager = aPerson.department.manager;</code></pre>

  <p class="zw">这样的编码就对客户端揭露了<code>Department</code>的工作原理，于是客户知道：<code>Department</code>负责追踪“经理”这条信息。如果对客户隐藏<code>Department</code>，可以减少耦合。为了这一目的，我在<code>Person</code>中建立一个简单的委托函数。</p>

  <h5 class="sigil_not_in_toc">class Person...</h5>
  <pre class="代码无行号"><code>get manager() {return this._department.manager;}</code></pre>

  <p class="zw">现在，我得修改<code>Person</code>的所有客户端，让它们改用新函数：</p>

  <h5 class="sigil_not_in_toc">客户端代码...</h5>
  <pre class="代码无行号"><code>manager = aPerson<del>.department.</del>manager;</code></pre>

  <p class="zw">只要完成了对<code>Department</code>所有函数的修改，并相应修改了<code>Person</code>的所有客户端，我就可以移除<code>Person</code>中的<code>department</code>访问函数了。</p>

  <h2 id="nav_point_185">7.8　移除中间人（Remove Middle Man）</h2>

  <p class="zw">反向重构：<span style="">隐藏委托关系（189）</span></p>

  <p class="图"><img alt="" src="../Images/image00319.jpeg" style="width: 40%" width="40%"/></p>
  <pre class="代码无行号"><code>manager = aPerson.manager; 

class Person {
　get manager() {return this.department.manager;}</code></pre>

  <p class="图"><img alt="图像说明文字" src="../Images/image00320.jpeg" style="width: 7%" width="7%"/></p>
  <pre class="代码无行号"><code>manager = aPerson.department.manager;</code></pre>

  <h3 class="sigil_not_in_toc" id="nav_point_186">动机</h3>

  <p class="zw">在<span style="">隐藏委托关系（189）</span>的“动机”一节中，我谈到了“封装受托对象”的好处。但是这层封装也是有代价的。每当客户端要使用受托类的新特性时，你就必须在服务端添加一个简单委托函数。随着受托类的特性（功能）越来越多，更多的转发函数就会使人烦躁。服务类完全变成了一个<span style="">中间人（81）</span>，此时就应该让客户直接调用受托类。（这个味道通常在人们狂热地遵循迪米特法则时悄然出现。我总觉得，如果这条法则当初叫作“偶尔有用的迪米特建议”，如今能少很多烦恼。）</p>

  <p class="zw">很难说什么程度的隐藏才是合适的。还好，有了<span style="">隐藏委托关系（189）</span>和删除中间人，我大可不必操心这个问题，因为我可以在系统运行过程中不断进行调整。随着代码的变化，“合适的隐藏程度”这个尺度也相应改变。6个月前恰如其分的封装，现今可能就显得笨拙。重构的意义就在于：你永远不必说对不起——只要把出问题的地方修补好就行了。</p>

  <h3 class="sigil_not_in_toc" id="nav_point_187">做法</h3>

  <ul>
    <li class="第1级无序列表">为受托对象创建一个取值函数。</li>

    <li class="第1级无序列表">对于每个委托函数，让其客户端转为连续的访问函数调用。每次替换后运行测试。</li>
  </ul>

  <blockquote>
    <p class="zw">替换完委托方法的所有调用点后，你就可以删掉这个委托方法了。</p>

    <p class="zw">这能通过可自动化的重构手法来完成，你可以先对受托字段使用<span style="">封装变量（132）</span>，再应用<span style="">内联函数（115）</span>内联所有使用它的函数。</p>
  </blockquote>

  <h3 class="sigil_not_in_toc" id="nav_point_188">范例</h3>

  <p class="zw">我又要从一个<code>Person</code>类开始了，这个类通过维护一个部门对象来决定某人的经理是谁。（如果你一口气读完本书的好几章，可能会发现每个“人与部门”的例子都出奇地相似。）</p>

  <h5 class="sigil_not_in_toc">客户端代码...</h5>
  <pre class="代码无行号"><code>manager = aPerson.manager;</code></pre>

  <h5 class="sigil_not_in_toc">class Person...</h5>
  <pre class="代码无行号"><code>get manager() {return this._department.manager;}</code></pre>

  <h5 class="sigil_not_in_toc">class Department...</h5>
  <pre class="代码无行号"><code>get manager() {return this._manager;}</code></pre>

  <p class="zw">像这样，使用和封装<code>Department</code>都很简单。但如果大量函数都这么做，我就不得不在<code>Person</code>之中安置大量委托行为。这就该是移除中间人的时候了。首先在<code>Person</code>中建立一个函数，用于获取受托对象。</p>

  <h5 class="sigil_not_in_toc">class Person...</h5>
  <pre class="代码无行号"><code>get department() {return this._department;}</code></pre>

  <p class="zw">然后逐一处理每个客户端，使它们直接通过受托对象完成工作。</p>

  <h5 class="sigil_not_in_toc">客户端代码...</h5>
  <pre class="代码无行号"><code>manager = aPerson.department.manager;</code></pre>

  <p class="zw">完成对客户端引用点的替换后，我就可以从<code>Person</code>中移除<code>manager</code>方法了。我可以重复此法，移除<code>Person</code>中其他类似的简单委托函数。</p>

  <p class="zw">我可以混用两种用法。有些委托关系非常常用，因此我想保住它们，这样可使客户端代码调用更友好。何时应该隐藏委托关系，何时应该移除中间人，对我而言没有绝对的标准——代码环境自然会给出该使用哪种手法的线索，具备思考能力的程序员应能分辨出何种手法更佳。</p>

  <p class="zw">如果手边在用自动化的重构工具，那么本手法的步骤有一个实用的变招：我可以先对<code>department</code>应用<span style="">封装变量（132）</span>。这样可让<code>manager</code>的取值函数调用<code>department</code>的取值函数。</p>

  <h5 class="sigil_not_in_toc">class Person...</h5>
  <pre class="代码无行号"><code>get manager() {return this.department.manager;}</code></pre>

  <p class="zw"><span style="">在JavaScript中，调用取值函数的语法跟取用普通字段看起来很像，但通过移除<code>department</code>字段的下划线，我想表达出这里是调用了取值函数而非直接取用字段的区别。</span></p>

  <p class="zw">然后我对<code>manager</code>方法应用<span style="">内联函数（115）</span>，一口气替换它的所有调用点。</p>

  <h2 id="nav_point_189">7.9　替换算法（Substitute Algorithm）</h2>

  <p class="图"><img alt="" src="../Images/image00321.jpeg" style="width: 40%" width="40%"/></p>
  <pre class="代码无行号"><code>function foundPerson(people) {
　for(let i = 0; i &lt; people.length; i++) { 
　　if (people[i] === "Don") {
　　　return "Don";
　　}
　　if (people[i] === "John") { 
　　　return "John";
　　}
　　if (people[i] === "Kent") { 
　　　return "Kent";
　　}
　}
　return "";
}</code></pre>

  <p class="图"><img alt="图像说明文字" src="../Images/image00322.jpeg" style="width: 7%" width="7%"/></p>
  <pre class="代码无行号"><code>function foundPerson(people) {
　const candidates = ["Don", "John", "Kent"];
　return people.find(p =&gt; candidates.includes(p)) || '';
}</code></pre>

  <h3 class="sigil_not_in_toc" id="nav_point_190">动机</h3>

  <p class="zw">我从没试过给猫剥皮，听说有好几种方法，我敢肯定，其中某些方法会比另一些简单。算法也是如此。如果我发现做一件事可以有更清晰的方式，我就会用比较清晰的方式取代复杂的方式。“重构”可以把一些复杂的东西分解为较简单的小块，但有时你就必须壮士断腕，删掉整个算法，代之以较简单的算法。随着对问题有了更多理解，我往往会发现，在原先的做法之外，有更简单的解决方案，此时我就需要改变原先的算法。如果我开始使用程序库，而其中提供的某些功能/特性与我自己的代码重复，那么我也需要改变原先的算法。</p>

  <p class="zw">有时我会想修改原先的算法，让它去做一件与原先略有差异的事。这时候可以先把原先的算法替换为一个较易修改的算法，这样后续的修改会轻松许多。</p>

  <p class="zw">使用这项重构手法之前，我得确定自己已经尽可能分解了原先的函数。替换一个巨大且复杂的算法是非常困难的，只有先将它分解为较简单的小型函数，我才能很有把握地进行算法替换工作。</p>

  <h3 class="sigil_not_in_toc" id="nav_point_191">做法</h3>

  <ul>
    <li class="第1级无序列表">整理一下待替换的算法，保证它已经被抽取到一个独立的函数中。</li>

    <li class="第1级无序列表">先只为这个函数准备测试，以便固定它的行为。</li>

    <li class="第1级无序列表">准备好另一个（替换用）算法。</li>

    <li class="第1级无序列表">执行静态检查。</li>

    <li class="第1级无序列表">运行测试，比对新旧算法的运行结果。如果测试通过，那就大功告成；否则，在后续测试和调试过程中，以旧算法为比较参照标准。</li>
  </ul>

  <p class="zw"><br style="page-break-after:always"/><div style="page-break-after:always"></div></p>
</body></html>