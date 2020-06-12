Elasticsearch 拼音分析插件
==================================

这个拼音分析插件是用来做汉字和拼音之间的转换，该插件集成了NLP工具 (https://github.com/NLPchina/nlp-lang).

    --------------------------------------------------
    | Pinyin Analysis Plugin        | Elasticsearch  |
    --------------------------------------------------
    | master                        | 7.x -> master  |
    --------------------------------------------------
    | 6.x                           | 6.x            |
    --------------------------------------------------  
    | 5.x                           | 5.x            |
    --------------------------------------------------  
    | 1.8.1                         | 2.4.1          |
    --------------------------------------------------  
    | 1.7.5                         | 2.3.5          |
    --------------------------------------------------  
    | 1.6.1                         | 2.2.1          |
    --------------------------------------------------
    | 1.5.0                         | 2.1.0          |
    --------------------------------------------------
    | 1.4.0                         | 2.0.x          |
    --------------------------------------------------
    | 1.3.0                         | 1.6.x          |
    --------------------------------------------------
    | 1.2.2                         | 1.0.x          |
    --------------------------------------------------

该插件提供了analyzer: `pinyin` ,  tokenizer: `pinyin` 和 token-filter:  `pinyin`.

** 可选参数 ** 
* `keep_first_letter` 当这个选项启用, 会生成拼音首字母组合例如： `刘德华`>`ldh`, 默认： true
* `keep_separate_first_letter` 当这个选项启用, 会使用分隔符`,`储存拼音首字母组合,  例如： `刘德华`>`l`,`d`,`h`, 默认：false, 注意： 查询结果可能会因为语义单元(term)过于频繁而变得模糊
* `limit_first_letter_length` 设置拼音首字母组合的最大长度, 默认： 16
* `keep_full_pinyin` 当这个选项启用, 会生成全拼音组合例如： `刘德华`> [`liu`,`de`,`hua`], 默认： true
* `keep_joined_full_pinyin` 当这个选项启用，会不含分隔符`,`生成全拼音例如： `刘德华`> [`liudehua`], 默认：false
* `keep_none_chinese` 在结果中保留非中文拼音字母和数字， 默认： true
* `keep_none_chinese_together` 保持非中文拼音字母组合在一起， 默认： true, 例如： `DJ音乐家` -> `DJ`,`yin`,`yue`,`jia`, 如果设定成 `false`, 例如： `DJ音乐家` -> `D`,`J`,`yin`,`yue`,`jia`, 注意： `keep_none_chinese` 需要先启用
* `keep_none_chinese_in_first_letter` 拼音首字母组合缩写中保留非中文拼音字母, 例如： `刘德华AT2016`->`ldhat2016`, 默认： true
* `keep_none_chinese_in_joined_full_pinyin` 拼音组合中保留非中文拼音字母, 例如： `刘德华2016`->`liudehua2016`, 默认：false
* `none_chinese_pinyin_tokenize` 会将非中文拼音字母也以中文拼音的方式进行分割成语义单元(term), 默认： true, 例如： `liudehuaalibaba13zhuanghan` -> `liu`,`de`,`hua`,`a`,`li`,`ba`,`ba`,`13`,`zhuang`,`han`, 注意：  `keep_none_chinese` 和 `keep_none_chinese_together` 需要先启用
* `keep_original` 当这个选项启用, 原始输入也会保留, 默认：false
* `lowercase`  将非中文拼音字母转为小写, 默认： true
* `trim_whitespace` 默认： true
* `remove_duplicated_term` 当这个选项启用, 重复的语义单元(term)会被移除以节省索引， 例如： `de的`>`de`, 默认：false,  注意： 基于位置相关的查询(position related query)可能会受到影响
* `ignore_pinyin_offset` 6.0之后, 偏移量被严格约束, overlapped tokens不允许使用，使用这个参数时， overlapped token 在忽略偏移量之后可以使用，请注意，所有的位置相关的查询(position related query)或者高亮都会变得不正确，针对不同的查询目标应该使用multi fields和指定不同的设置。如果需要偏移量，设为false。 默认：true.



1.创建一个带有pinyin analyzer的索引
<pre>
PUT /medcl/ 
{
    "settings" : {
        "analysis" : {
            "analyzer" : {
                "pinyin_analyzer" : {
                    "tokenizer" : "my_pinyin"
                    }
            },
            "tokenizer" : {
                "my_pinyin" : {
                    "type" : "pinyin",
                    "keep_separate_first_letter" : false,
                    "keep_full_pinyin" : true,
                    "keep_original" : true,
                    "limit_first_letter_length" : 16,
                    "lowercase" : true,
                    "remove_duplicated_term" : true
                }
            }
        }
    }
}
</pre>

2.测试 Analyzer, 分析一个中午姓名, 如：刘德华
<pre>
GET /medcl/_analyze
{
  "text": ["刘德华"],
  "analyzer": "pinyin_analyzer"
}</pre>
<pre>
{
  "tokens" : [
    {
      "token" : "liu",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "de",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "hua",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "刘德华",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "word",
      "position" : 3
    },
    {
      "token" : "ldh",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "word",
      "position" : 4
    }
  ]
}
</pre>

3.创建 mapping
<pre>
POST /medcl/_mapping 
{
        "properties": {
            "name": {
                "type": "keyword",
                "fields": {
                    "pinyin": {
                        "type": "text",
                        "store": false,
                        "term_vector": "with_offsets",
                        "analyzer": "pinyin_analyzer",
                        "boost": 10
                    }
                }
            }
        }
    
}
</pre>

4.Indexing
<pre>
POST /medcl/_create/andy
{"name":"刘德华"}
</pre>

5.看下搜索情况

<pre>

curl http://localhost:9200/medcl/_search?q=name:%E5%88%98%E5%BE%B7%E5%8D%8E
curl http://localhost:9200/medcl/_search?q=name.pinyin:%e5%88%98%e5%be%b7
curl http://localhost:9200/medcl/_search?q=name.pinyin:liu
curl http://localhost:9200/medcl/_search?q=name.pinyin:ldh
curl http://localhost:9200/medcl/_search?q=name.pinyin:de+hua

</pre>

6.使用 Pinyin-TokenFilter
<pre>
PUT /medcl1/ 
{
    "settings" : {
        "analysis" : {
            "analyzer" : {
                "user_name_analyzer" : {
                    "tokenizer" : "whitespace",
                    "filter" : "pinyin_first_letter_and_full_pinyin_filter"
                }
            },
            "filter" : {
                "pinyin_first_letter_and_full_pinyin_filter" : {
                    "type" : "pinyin",
                    "keep_first_letter" : true,
                    "keep_full_pinyin" : false,
                    "keep_none_chinese" : true,
                    "keep_original" : false,
                    "limit_first_letter_length" : 16,
                    "lowercase" : true,
                    "trim_whitespace" : true,
                    "keep_none_chinese_in_first_letter" : true
                }
            }
        }
    }
}
</pre>

Token Test:刘德华 张学友 郭富城 黎明 四大天王
<pre>
GET /medcl1/_analyze
{
  "text": ["刘德华 张学友 郭富城 黎明 四大天王"],
  "analyzer": "user_name_analyzer"
}
</pre>
<pre>
{
  "tokens" : [
    {
      "token" : "ldh",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "zxy",
      "start_offset" : 4,
      "end_offset" : 7,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "gfc",
      "start_offset" : 8,
      "end_offset" : 11,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "lm",
      "start_offset" : 12,
      "end_offset" : 14,
      "type" : "word",
      "position" : 3
    },
    {
      "token" : "sdtw",
      "start_offset" : 15,
      "end_offset" : 19,
      "type" : "word",
      "position" : 4
    }
  ]
}
</pre>


7.在短语查询中使用

- option 1

<pre>
PUT /medcl2/
{
    "settings" : {
        "analysis" : {
            "analyzer" : {
                "pinyin_analyzer" : {
                    "tokenizer" : "my_pinyin"
                    }
            },
            "tokenizer" : {
                "my_pinyin" : {
                    "type" : "pinyin",
                    "keep_first_letter":false,
                    "keep_separate_first_letter" : false,
                    "keep_full_pinyin" : true,
                    "keep_original" : false,
                    "limit_first_letter_length" : 16,
                    "lowercase" : true
                }
            }
        }
    }
}
GET /medcl2/_search
{
  "query": {"match_phrase": {
    "name.pinyin": "刘德华"
  }}
}

</pre>

- option 2

<pre>
 
PUT /medcl3/
{
   "settings" : {
       "analysis" : {
           "analyzer" : {
               "pinyin_analyzer" : {
                   "tokenizer" : "my_pinyin"
                   }
           },
           "tokenizer" : {
               "my_pinyin" : {
                   "type" : "pinyin",
                   "keep_first_letter":true,
                   "keep_separate_first_letter" : true,
                   "keep_full_pinyin" : true,
                   "keep_original" : false,
                   "limit_first_letter_length" : 16,
                   "lowercase" : true
               }
           }
       }
   }
}
   
POST /medcl3/_mapping 
{
  "properties": {
      "name": {
          "type": "keyword",
          "fields": {
              "pinyin": {
                  "type": "text",
                  "store": false,
                  "term_vector": "with_offsets",
                  "analyzer": "pinyin_analyzer",
                  "boost": 10
              }
          }
      }
  }
}
  
   
GET /medcl3/_analyze
{
   "text": ["刘德华"],
   "analyzer": "pinyin_analyzer"
}
 
POST /medcl3/_create/andy
{"name":"刘德华"}

GET /medcl3/_search
{
 "query": {"match_phrase": {
   "name.pinyin": "刘德h"
 }}
}

GET /medcl3/_search
{
 "query": {"match_phrase": {
   "name.pinyin": "刘dh"
 }}
}

GET /medcl3/_search
{
 "query": {"match_phrase": {
   "name.pinyin": "liudh"
 }}
}

GET /medcl3/_search
{
 "query": {"match_phrase": {
   "name.pinyin": "liudeh"
 }}
}

GET /medcl3/_search
{
 "query": {"match_phrase": {
   "name.pinyin": "liude华"
 }}
}

</pre>

8.谢谢观看.
