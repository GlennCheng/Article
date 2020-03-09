---


---

<h1 id="如何在magento2中的searchcriteria-filter加入or條件">如何在Magento2中的searchCriteria filter加入"OR"條件</h1>
<p>在建立module或客製化功能時，一定常常都會需要對資料庫操作，其中也不免的會需要下各種的搜尋條件，不論是單純的一個where，抑或是OR、AND，再搭配其他語法達到自己想要的結果。<br>
那相信會點到這篇文章的各位，一定是對於M2中要如何使用"OR"條件感到有所疑惑， 當然大家可以直接編寫raw sql來query，不過以下我們還是以M2的ORM來說明及操作。</p>
<h2 id="測試實作">測試實作</h2>
<p>我們會以會員當作例子並使用customerRepository及customerRepository的getList()方法傳入searchCriteria來做測試。</p>
<h3 id="前置作業">前置作業</h3>
<p>由於Repository並不像Collection可以直接getSelect()拿到語法，不過我們知道Repository底層也是Collection的操作˙所以我們先將customerRepository中的getList()方法稍微修改一下讓他可以直接丟出Collection的MySQL query語法。</p>
<pre class=" language-php"><code class="prism  language-php"><span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">getList</span><span class="token punctuation">(</span>SearchCriteriaInterface <span class="token variable">$searchCriteria</span><span class="token punctuation">)</span>  
<span class="token punctuation">{</span>
	 <span class="token variable">$searchResults</span> <span class="token operator">=</span> <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">searchResultsFactory</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
	 <span class="token variable">$searchResults</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setSearchCriteria</span><span class="token punctuation">(</span><span class="token variable">$searchCriteria</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
  <span class="token comment">/** @var \Magento\Customer\Model\ResourceModel\Customer\Collection $collection */</span>  
	 <span class="token variable">$collection</span> <span class="token operator">=</span> <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">customerFactory</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getCollection</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
	<span class="token comment">//        $this-&gt;extensionAttributesJoinProcessor-&gt;process(  </span>
	<span class="token comment">//            $collection,  </span>
	<span class="token comment">//            CustomerInterface::class  </span>
	<span class="token comment">//        );  </span>
	<span class="token comment">//        // This is needed to make sure all the attributes are properly loaded  </span>
	<span class="token comment">//        foreach ($this-&gt;customerMetadata-&gt;getAllAttributesMetadata() as $metadata) {  </span>
	<span class="token comment">//            $collection-&gt;addAttributeToSelect($metadata-&gt;getAttributeCode());  </span>
	<span class="token comment">//        }  </span>
	<span class="token comment">//        // Needed to enable filtering on name as a whole  </span>
	<span class="token comment">//        $collection-&gt;addNameToSelect();  </span>
	<span class="token comment">//        // Needed to enable filtering based on billing address attributes  </span>
	<span class="token comment">//        $collection-&gt;joinAttribute('billing_postcode', 'customer_address/postcode', 'default_billing', null, 'left')  </span>
	<span class="token comment">//            -&gt;joinAttribute('billing_city', 'customer_address/city', 'default_billing', null, 'left')  </span>
	<span class="token comment">//            -&gt;joinAttribute('billing_telephone', 'customer_address/telephone', 'default_billing', null, 'left')  </span>
	<span class="token comment">//            -&gt;joinAttribute('billing_region', 'customer_address/region', 'default_billing', null, 'left')  </span>
	<span class="token comment">//            -&gt;joinAttribute('billing_country_id', 'customer_address/country_id', 'default_billing', null, 'left')  </span>
	<span class="token comment">//            -&gt;joinAttribute('billing_company', 'customer_address/company', 'default_billing', null, 'left');  </span>
	 
	 <span class="token comment">//以上是將customer相關表格及extensionAttributes加入語法中，不過我們為了簡單所以全部註解掉</span>
	 <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">collectionProcessor</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">process</span><span class="token punctuation">(</span><span class="token variable">$searchCriteria</span><span class="token punctuation">,</span> <span class="token variable">$collection</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
	 
	 <span class="token keyword">echo</span> <span class="token variable">$collection</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getSelect</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>exit<span class="token punctuation">;</span> <span class="token comment">//主要加上這一行丟出語法</span>
<span class="token punctuation">}</span>
</code></pre>
<h2 id="實際測試">實際測試</h2>
<p>OK，稍微調整了repository的getList()方法後，現在再來我們來試試並比較一下以下兩種結果</p>
<h3 id="searchcriteriabuilder-addfilter">searchCriteriaBuilder-&gt;addFilter</h3>
<pre class=" language-php"><code class="prism  language-php"><span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">__construct</span><span class="token punctuation">(</span>  
	 Session <span class="token variable">$session</span><span class="token punctuation">,</span>  
	 Context <span class="token variable">$context</span><span class="token punctuation">,</span>  
	 SearchCriteriaBuilder <span class="token variable">$searchCriteriaBuilder</span><span class="token punctuation">,</span>  
	 \<span class="token package">Magento<span class="token punctuation">\</span>Customer<span class="token punctuation">\</span>Api<span class="token punctuation">\</span>CustomerRepositoryInterface</span> <span class="token variable">$customerRepository</span><span class="token punctuation">,</span>  
	 \<span class="token package">Magento<span class="token punctuation">\</span>Framework<span class="token punctuation">\</span>Api<span class="token punctuation">\</span>FilterBuilder</span> <span class="token variable">$filterBuilder</span><span class="token punctuation">,</span>  
	 \<span class="token package">Magento<span class="token punctuation">\</span>Framework<span class="token punctuation">\</span>Api<span class="token punctuation">\</span>Search<span class="token punctuation">\</span>FilterGroupBuilder</span> <span class="token variable">$filterGroupBuilder</span>  
<span class="token punctuation">)</span> <span class="token punctuation">{</span>  
	 <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">session</span> <span class="token operator">=</span> <span class="token variable">$session</span><span class="token punctuation">;</span>  
	 <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">customerRepository</span> <span class="token operator">=</span> <span class="token variable">$customerRepository</span><span class="token punctuation">;</span>  
	 <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">searchCriteriaBuilder</span> <span class="token operator">=</span> <span class="token variable">$searchCriteriaBuilder</span><span class="token punctuation">;</span>  
	 <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">filterBuilder</span> <span class="token operator">=</span> <span class="token variable">$filterBuilder</span><span class="token punctuation">;</span>  
	 <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">filterGroupBuilder</span> <span class="token operator">=</span> <span class="token variable">$filterGroupBuilder</span><span class="token punctuation">;</span>  
	 <span class="token keyword">return</span> <span class="token keyword">parent</span><span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">__construct</span><span class="token punctuation">(</span><span class="token variable">$context</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
<span class="token punctuation">}</span>

<span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">execute</span><span class="token punctuation">(</span><span class="token punctuation">)</span>  
<span class="token punctuation">{</span>
	<span class="token variable">$searchCriteria</span> <span class="token operator">=</span> <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">searchCriteriaBuilder</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">addFilter</span><span class="token punctuation">(</span><span class="token string">'entity_id'</span><span class="token punctuation">,</span> <span class="token number">1</span><span class="token punctuation">,</span> <span class="token string">'eq'</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">addFilter</span><span class="token punctuation">(</span><span class="token string">'entity_id'</span><span class="token punctuation">,</span> <span class="token number">2</span><span class="token punctuation">,</span> <span class="token string">'eq'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

	<span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">customerRepository</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getList</span><span class="token punctuation">(</span><span class="token variable">$searchCriteria</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	exit<span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<p>結果</p>
<pre class=" language-sql"><code class="prism  language-sql"><span class="token keyword">SELECT</span> <span class="token punctuation">`</span><span class="token number">e</span><span class="token punctuation">`</span><span class="token punctuation">.</span><span class="token operator">*</span> <span class="token keyword">FROM</span> <span class="token punctuation">`</span>customer_entity<span class="token punctuation">`</span> <span class="token keyword">AS</span> <span class="token punctuation">`</span><span class="token number">e</span><span class="token punctuation">`</span> <span class="token keyword">WHERE</span> <span class="token punctuation">(</span><span class="token punctuation">(</span><span class="token punctuation">`</span><span class="token number">e</span><span class="token punctuation">`</span><span class="token punctuation">.</span><span class="token punctuation">`</span>entity_id<span class="token punctuation">`</span> <span class="token operator">=</span> <span class="token number">1</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token operator">AND</span> <span class="token punctuation">(</span><span class="token punctuation">(</span><span class="token punctuation">`</span><span class="token number">e</span><span class="token punctuation">`</span><span class="token punctuation">.</span><span class="token punctuation">`</span>entity_id<span class="token punctuation">`</span> <span class="token operator">=</span> <span class="token number">2</span><span class="token punctuation">)</span><span class="token punctuation">)</span>
</code></pre>
<p>這邊我們可以看到sql語法中間是用"AND"來連接</p>
<hr>
<h3 id="searchcriteriabuilder-setfiltergroups">searchCriteriaBuilder-&gt;setFilterGroups</h3>
<pre class=" language-php"><code class="prism  language-php"><span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">__construct</span><span class="token punctuation">(</span>  
	 Session <span class="token variable">$session</span><span class="token punctuation">,</span>  
	 Context <span class="token variable">$context</span><span class="token punctuation">,</span>  
	 PageFactory <span class="token variable">$resultPageFactory</span><span class="token punctuation">,</span>  
	 ApiResourceInterface <span class="token variable">$apiResource</span><span class="token punctuation">,</span>  
	 SearchCriteriaBuilder <span class="token variable">$searchCriteriaBuilder</span><span class="token punctuation">,</span>  
	 \<span class="token package">Magento<span class="token punctuation">\</span>Customer<span class="token punctuation">\</span>Api<span class="token punctuation">\</span>CustomerRepositoryInterface</span> <span class="token variable">$customerRepository</span><span class="token punctuation">,</span>  
	 \<span class="token package">Magento<span class="token punctuation">\</span>Framework<span class="token punctuation">\</span>Api<span class="token punctuation">\</span>FilterBuilder</span> <span class="token variable">$filterBuilder</span><span class="token punctuation">,</span>  
	 \<span class="token package">Magento<span class="token punctuation">\</span>Framework<span class="token punctuation">\</span>Api<span class="token punctuation">\</span>Search<span class="token punctuation">\</span>FilterGroupBuilder</span> <span class="token variable">$filterGroupBuilder</span>  
<span class="token punctuation">)</span> <span class="token punctuation">{</span>  
	 <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">session</span> <span class="token operator">=</span> <span class="token variable">$session</span><span class="token punctuation">;</span>  
	 <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">apiResource</span> <span class="token operator">=</span> <span class="token variable">$apiResource</span><span class="token punctuation">;</span>  
	 <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">resultPageFactory</span> <span class="token operator">=</span> <span class="token variable">$resultPageFactory</span><span class="token punctuation">;</span>  
	 <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">customerRepository</span> <span class="token operator">=</span> <span class="token variable">$customerRepository</span><span class="token punctuation">;</span>  
	 <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">searchCriteriaBuilder</span> <span class="token operator">=</span> <span class="token variable">$searchCriteriaBuilder</span><span class="token punctuation">;</span>  
	 <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">filterBuilder</span> <span class="token operator">=</span> <span class="token variable">$filterBuilder</span><span class="token punctuation">;</span>  
	 <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">filterGroupBuilder</span> <span class="token operator">=</span> <span class="token variable">$filterGroupBuilder</span><span class="token punctuation">;</span>  
	 <span class="token keyword">return</span> <span class="token keyword">parent</span><span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">__construct</span><span class="token punctuation">(</span><span class="token variable">$context</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
<span class="token punctuation">}</span>

<span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">execute</span><span class="token punctuation">(</span><span class="token punctuation">)</span>  
<span class="token punctuation">{</span>
    <span class="token variable">$entityFilter</span> <span class="token operator">=</span> <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">filterBuilder</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setField</span><span class="token punctuation">(</span><span class="token string">'entity_id'</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setConditionType</span><span class="token punctuation">(</span><span class="token string">'eq'</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setValue</span><span class="token punctuation">(</span><span class="token number">1</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
  
    <span class="token variable">$entityFilter2</span> <span class="token operator">=</span> <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">filterBuilder</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setField</span><span class="token punctuation">(</span><span class="token string">'entity_id'</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setConditionType</span><span class="token punctuation">(</span><span class="token string">'eq'</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setValue</span><span class="token punctuation">(</span><span class="token number">2</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
  
    <span class="token variable">$filterGroup1</span> <span class="token operator">=</span> <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">filterGroupBuilder</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">addFilter</span><span class="token punctuation">(</span><span class="token variable">$entityFilter</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">addFilter</span><span class="token punctuation">(</span><span class="token variable">$entityFilter2</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
  
    <span class="token variable">$searchCriteria</span> <span class="token operator">=</span> <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">searchCriteriaBuilder</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setFilterGroups</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token variable">$filterGroup1</span><span class="token punctuation">]</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">customerRepository</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getList</span><span class="token punctuation">(</span><span class="token variable">$searchCriteria</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	exit<span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<p>結果</p>
<pre class=" language-sql"><code class="prism  language-sql"><span class="token keyword">SELECT</span> <span class="token punctuation">`</span><span class="token number">e</span><span class="token punctuation">`</span><span class="token punctuation">.</span><span class="token operator">*</span> <span class="token keyword">FROM</span> <span class="token punctuation">`</span>customer_entity<span class="token punctuation">`</span> <span class="token keyword">AS</span> <span class="token punctuation">`</span><span class="token number">e</span><span class="token punctuation">`</span> <span class="token keyword">WHERE</span> <span class="token punctuation">(</span><span class="token punctuation">(</span><span class="token punctuation">`</span><span class="token number">e</span><span class="token punctuation">`</span><span class="token punctuation">.</span><span class="token punctuation">`</span>entity_id<span class="token punctuation">`</span> <span class="token operator">=</span> <span class="token number">1</span><span class="token punctuation">)</span> <span class="token operator">OR</span> <span class="token punctuation">(</span><span class="token punctuation">`</span><span class="token number">e</span><span class="token punctuation">`</span><span class="token punctuation">.</span><span class="token punctuation">`</span>entity_id<span class="token punctuation">`</span> <span class="token operator">=</span> <span class="token number">2</span><span class="token punctuation">)</span><span class="token punctuation">)</span>
</code></pre>
<p>可以看到，這時候where條件用的是"OR"。</p>
<p>且我們還發現OR左右的兩個條件，外層還有一個括號把兩個條件包在一起，<br>
整個語法邏輯符合且對應程式碼中先設定兩個Filter並組合成一個Filter Group。</p>
<hr>
<p>這時候一定有人發現</p>
<pre class=" language-php"><code class="prism  language-php"> <span class="token variable">$searchCriteria</span> <span class="token operator">=</span> <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">searchCriteriaBuilder</span>  
    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setFilterGroups</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token variable">$filterGroup1</span><span class="token punctuation">]</span><span class="token punctuation">)</span>  
    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<p>這裡丟進setFilterGroup去的是array型別，那如果我們在多塞一個在array中呢？<br>
我們現在就來實際測試看看：</p>
<pre class=" language-php"><code class="prism  language-php"><span class="token keyword">public</span> <span class="token keyword">function</span> <span class="token function">execute</span><span class="token punctuation">(</span><span class="token punctuation">)</span>  
<span class="token punctuation">{</span>
	<span class="token comment">//group1</span>
    <span class="token variable">$entityFilter</span> <span class="token operator">=</span> <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">filterBuilder</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setField</span><span class="token punctuation">(</span><span class="token string">'entity_id'</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setConditionType</span><span class="token punctuation">(</span><span class="token string">'eq'</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setValue</span><span class="token punctuation">(</span><span class="token number">1</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
  
    <span class="token variable">$entityFilter2</span> <span class="token operator">=</span> <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">filterBuilder</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setField</span><span class="token punctuation">(</span><span class="token string">'entity_id'</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setConditionType</span><span class="token punctuation">(</span><span class="token string">'eq'</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setValue</span><span class="token punctuation">(</span><span class="token number">2</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
    
    <span class="token variable">$filterGroup1</span> <span class="token operator">=</span> <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">filterGroupBuilder</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">addFilter</span><span class="token punctuation">(</span><span class="token variable">$entityFilter</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">addFilter</span><span class="token punctuation">(</span><span class="token variable">$entityFilter2</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
	    
	<span class="token comment">//group2</span>
	<span class="token variable">$websiteFilter</span> <span class="token operator">=</span> <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">filterBuilder</span>  
		<span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setField</span><span class="token punctuation">(</span><span class="token string">'website_id'</span><span class="token punctuation">)</span>  
		<span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setConditionType</span><span class="token punctuation">(</span><span class="token string">'eq'</span><span class="token punctuation">)</span>  
		<span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setValue</span><span class="token punctuation">(</span><span class="token number">1</span><span class="token punctuation">)</span>  
		<span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
  
	<span class="token variable">$storeFilter</span> <span class="token operator">=</span> <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">filterBuilder</span>  
		<span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setField</span><span class="token punctuation">(</span><span class="token string">'store_id'</span><span class="token punctuation">)</span>  
		<span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setConditionType</span><span class="token punctuation">(</span><span class="token string">'eq'</span><span class="token punctuation">)</span>  
		<span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setValue</span><span class="token punctuation">(</span><span class="token number">1</span><span class="token punctuation">)</span>  
		<span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  

	<span class="token variable">$filterGroup2</span> <span class="token operator">=</span> <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">filterGroupBuilder</span>  
		<span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">addFilter</span><span class="token punctuation">(</span><span class="token variable">$websiteFilter</span><span class="token punctuation">)</span>  
		<span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">addFilter</span><span class="token punctuation">(</span><span class="token variable">$storeFilter</span><span class="token punctuation">)</span>  
		<span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  
    <span class="token variable">$searchCriteria</span> <span class="token operator">=</span> <span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">searchCriteriaBuilder</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setFilterGroups</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token variable">$filterGroup1</span><span class="token punctuation">,</span> <span class="token variable">$filterGroup2</span><span class="token punctuation">]</span><span class="token punctuation">)</span>  
	    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token variable">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">customerRepository</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getList</span><span class="token punctuation">(</span><span class="token variable">$searchCriteria</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	exit<span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<p>結果</p>
<pre class=" language-sql"><code class="prism  language-sql"><span class="token keyword">SELECT</span> <span class="token punctuation">`</span><span class="token number">e</span><span class="token punctuation">`</span><span class="token punctuation">.</span><span class="token operator">*</span> <span class="token keyword">FROM</span> <span class="token punctuation">`</span>customer_entity<span class="token punctuation">`</span> <span class="token keyword">AS</span> <span class="token punctuation">`</span><span class="token number">e</span><span class="token punctuation">`</span> <span class="token keyword">WHERE</span> <span class="token punctuation">(</span><span class="token punctuation">(</span><span class="token punctuation">`</span><span class="token number">e</span><span class="token punctuation">`</span><span class="token punctuation">.</span><span class="token punctuation">`</span>entity_id<span class="token punctuation">`</span> <span class="token operator">=</span> <span class="token number">1</span><span class="token punctuation">)</span> <span class="token operator">OR</span> <span class="token punctuation">(</span><span class="token punctuation">`</span><span class="token number">e</span><span class="token punctuation">`</span><span class="token punctuation">.</span><span class="token punctuation">`</span>entity_id<span class="token punctuation">`</span> <span class="token operator">=</span> <span class="token number">2</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token operator">AND</span> <span class="token punctuation">(</span><span class="token punctuation">(</span><span class="token punctuation">`</span><span class="token number">e</span><span class="token punctuation">`</span><span class="token punctuation">.</span><span class="token punctuation">`</span>website_id<span class="token punctuation">`</span> <span class="token operator">=</span> <span class="token number">1</span><span class="token punctuation">)</span> <span class="token operator">OR</span> <span class="token punctuation">(</span><span class="token punctuation">`</span><span class="token number">e</span><span class="token punctuation">`</span><span class="token punctuation">.</span><span class="token punctuation">`</span>store_id<span class="token punctuation">`</span> <span class="token operator">=</span> <span class="token number">1</span><span class="token punctuation">)</span><span class="token punctuation">)</span>
</code></pre>
<p>很聰明的你們一定也猜到了，他會把每個Group之間用AND來連接</p>
<hr>
<p>現在就快把以上的範例實際運用在各位的實際情境中吧！<br>
最後，別忘了把在前置作業時customerRepository中，修改的getList()方法復原唷。</p>

