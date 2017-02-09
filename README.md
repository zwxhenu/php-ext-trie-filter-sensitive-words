php-ext-trie-filter-sensitive-words
========
使用php的trie_filter扩展，加词库过滤敏感词，trie_filter扩展是基于Double-Array Trie 树实现。具体算法可以百度。<br />
<h3>1、编译安装libdatrie</h3>
    tar -zvxf libdatrie-0.2.4.tar.gz
    cd libdatrie-0.2.4
    ./configure && make && make install 
<h3>2、下载php-ext-trie-filter(https://github.com/wulijun/php-ext-trie-filter)</h3>
    unzip php-ext-trie-filter-master.zip
    cd php-ext-trie-filter-master
    /usr/local/php/bin/phpize
    ./configure --with-php-config=/usr/local/php/bin/php-config
    make && make install 
 <h3>3、字典路径</h3>   
 <p>WORDSFILTER_TEXT_PATH 字典绝对路径例如：/var/www/html/dict.txt</P>
 <p>WORDSFILTER_BINARY_PATH 生成树文件 例如：/var/www/html/dict.tree</p>
<h3>4、下面php的伪代码</h3>
	/**
	* 创建敏感词字典树
	*/	
	public function create_tree()
	{
		$fp = fopen(WORDSFILTER_TEXT_PATH, "r");
		if(!fp)
		{
			return false;
		}
		// 初始化一个树
		$res_trie = trie_filter_new();
		while(!feof($fp))
		{
			$line = trim(fgets($fp));
			if(empty($line))
			{
				continue;
			}
			// 将敏感词插入树中
			trie_filter_store($res_trie, $line);
		}
		// 保存树
		trie_filter_save($res_trie, WORDSFILTER_BINARY_PATH);
		unset($res_trie);
		fclose($fp);
        }
	
	/**
	 * 过滤敏感词
	 */
	public function filter_content($content, $replace = '**')
	{
		// 加载敏感词库
		$res_trie = trie_filter_load(WORDSFILTER_BINARY_PATH);
		// 查询敏感词
		$arr_res = trie_filter_search_all($res_trie, $content);
		// 释放加载资源
		trie_filter_free($res_trie);
		if(is_array($arr_res))
		{
			foreach($arr_res as $k => $v)
			{
				$content = substr_replace($content, $replace, $v[0]);
			}
		}
		return $content;
	}
