# ES要点

* 倒排索引和正排索引：倒排索引是分词后的词条作为索引关键字，同时记录了哪些文档包含这些词条，正排索引是文档id指向词；
* 分词器：standard，simple，whitespace，stop，pattern，keyword（不分词）；
* ==倒排索引内包含==：文档id，词频TF，position词条在文档中出现的位置，用于检索，offset偏移量，用于高亮显示；
* 常见数据类型：keyword，text，long，integer，bool，object，date，ip等等
* keyword类型不分词，text类型分词；
* term和match：term不会对搜索条件分词，一般用于非文本字段查询，match会对搜索条件分词，一般用于文本字段查询；
* match和match_phrase：match是多个词条或的关系，match_phrase是多个词条且的关系，且对顺序敏感；
* slop搜索：slop是match phrase的参数，用于控制查询词项之间的位置偏移容忍度，但是也不能过大，过大会使查询结果偏离太多，对性能也有影响，最多5；
  * 允许词项不严格按顺序出现，如 ABC，bac，slop=2；
  * 允许词项中插入其他词，如AC，ABC，slop=1；
* ==multi match==：一个查询条件，查询多个字段
  * best_fields：查询结果包含任意查询条件，返回最佳匹配字段得分；
  * most_fields：查询结果包含任意查询条件，返回合并所有字段匹配得分；
  * cross_fields：跨字段匹配，and相当于多个字段整合为一个字段；
* bool查询
  * must&must_not：必须（不）匹配；
  * should：匹配should条件的一个或多个；
  * filter：过滤条件，不计算得分，只是提高查询效率；

* 分页：
  * from+size：适合小数据量，分页不深时使用（==原因==）
  * scroll：适合大数据量使用，但不适合实时。scroll会维持一个查询上下文，占用系统资源，如果有大量并发请求或长时间的请求，会占用大量系统资源。且初始化一个请求时，ES会创建一个搜索结果的快照，即使有文档更新或删除，scroll的结果都不会改变，所以不适合实时查询；
  * ==sort+search_after==：排序后，记录上一页最后一个排序值。只能从第一页开始查询，且不能跳页。

* ==集群==

