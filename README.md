# php-ext-trie-filter-sensitive-words
使用php的trie_filter扩展，加词库过滤敏感词，trie_filter扩展是基于Double-Array Trie 树实现。具体算法可以百度。<br />
<h3>1、编译安装libdatrie</h3>
    tar -zvxf libdatrie-0.2.4.tar.gz<br />
    cd libdatrie-0.2.4<br />
    ./configure && make && make install <br />
<h3>2、下载php-ext-trie-filter(https://github.com/wulijun/php-ext-trie-filter)</h3>
    unzip php-ext-trie-filter-master.zip<br />
    cd php-ext-trie-filter-master<br />
    /usr/local/php/bin/phpize<br />
    ./configure --with-php-config=/usr/local/php/bin/php-config<br />
    make && make install <br />
 <h3>3、字典路径</h3>   
 WORDSFILTER_TEXT_PATH 字典绝对路径例如：/var/www/html/dict.txt<br />
 WORDSFILTER_BINARY_PATH 生成树文件 例如：/var/www/html/dict.tree<br />
<h3>3、下面php的伪代码</h3>
        /*<br />
         * 创建敏感词字典树<br />
	 */<br />
	public function create_tree()<br />
	{<br />
		$fp = fopen(WORDSFILTER_TEXT_PATH, "r");<br />
		if(!fp)<br />
		{<br />
			return false;<br />
		}<br />
		// 初始化一个树<br />
		$res_trie = trie_filter_new();<br />
		while(!feof($fp))<br />
		{
			$line = trim(fgets($fp));<br />
			if(empty($line))<br />
			{<br />
				continue;<br />
			}<br />
			// 将敏感词插入树中<br />
			trie_filter_store($res_trie, $line);<br />
		}<br />
		// 保存树
		trie_filter_save($res_trie, WORDSFILTER_BINARY_PATH);<br />
		unset($res_trie);<br />
		fclose($fp);<br />
        }<br />
	
	/*<br />
	 * 过滤敏感词<br />
	 */<br />
	public function filter_content($content, $replace = '**')<br />
	{<br />
		// 加载敏感词库<br />
		$res_trie = trie_filter_load(WORDSFILTER_BINARY_PATH);<br />
		// 查询敏感词<br />
		$arr_res = trie_filter_search_all($res_trie, $content);<br />
		// 释放加载资源<br />
		trie_filter_free($res_trie);<br />
		if(is_array($arr_res))<br />
		{<br />
			foreach($arr_res as $k => $v)<br />
			{<br />
				$content = substr_replace($content, $replace, $v[0]);<br />
			}<br />
		}<br />
		return $content;<br />
	}<br />
