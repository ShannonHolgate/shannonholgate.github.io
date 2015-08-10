---
layout: post
title: "Hive XML Options"
description: "Quick look at current XML support using Hive"
category: hadoop
tags: [Hadoop, Hive, Impala, Big Data]
comments: true
---
I have been looking into the XML options for Hive metastore tables for the past couple of days and have a few thoughts to share on what's possible.

Hive
-------------------------

####<a name="opt1"></a>Option 1: Custom SerDe
The XML parsing options are quite similar to what we can achieve in Spark and MapReduce simply because the SerDe is written in Java anyway.
Here's an example of a table that we can create:

<pre><code class="language-sql">CREATE EXTERNAL TABLE TestTableXML (content STRING, link_id STRING)
ROW FORMAT SERDE 'com.ibm.spss.hive.serde2.xml.XmlSerDe'
WITH SERDEPROPERTIES (
	"column.xpath.content"="/",
	"column.xpath.link_id"="//*[local-name()='link_id']/text()"
)
STORED AS
	INPUTFORMAT 'com.ibm.spss.hive.serde2.xml.XmlInputFormat'
	OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.IgnoreKeyTextOutputFormat'
LOCATION "/user/shannonh/test-xml"
TBLPROPERTIES (
	"xmlinput.start"="<body ",
	"xmlinput.end"="</body&gt;"
);
</code></pre>

The parser used is: [XML-data-sources](https://github.com/dvasilen/Hive-XML-SerDe/wiki/XML-data-sources)

Instead of using the Xpath UDF we are using an XML SerDe, that way the tables can be queried through standard SQL. However, this diminishes the power of nested XML and requires a lot of foolery to get the table definition right.

####<a name="opt2"></a>Option 2: Hive XPath UDF
The other option would be to query the XML blob using the XPath UDF like so:
<pre><code class="language-sql">SELECT
xpath(content, "//*[local-name()='link_id']/text()")
FROM TestTableXML;</code></pre>

####Caveats

There are drawbacks in both instances when we need to use xml namespaces. In this example we have badly formatted XML documents with elements such as this:
<pre><code class="language-markup">&lt;Message xmlns="http://www.shannonholgate.com/not/real"></code></pre>

The namespace here should be <code class="language-markup">real</code>, although it is not defined and XPath won't pick it up as easy. This is why we are using the <code class="language-markup">local-name()</code> function in [Option 1](#opt1):
<pre><code class="language-sql">"column.xpath.correlation_id"="//*[local-name()='link_id']/text()"</code></pre>

Namespaces are not supported whatsoever in the Hive Xpath UDF, thus a custom UDF would need to be written.

Impala
-------------------------
Impala does **not** actually support custom SerDes so [Option 1](#opt1) isn't possible.

In contrast, it should be possible to use the Hive XPath UDF in [Option 2](#opt2). You just need to copy the hive-exec.jar into HDFS, then create a custom function:
<pre><code class="language-sql">CREATE FUNCTION xpath(string,string)
RETURNS string
LOCATION '/user/impala/hive-exec.jar' symbol='org.apache.hadoop.hive.ql.udf.xml.UDFXPathString';</code></pre>

This is a nice feature as Impala is written in C++. Sadly I have not been able to use this successfully plus, the debug and error messages are not very descriptive:

<pre><code class="language-markup">WARNINGS: ImpalaRuntimeException: Unable to call create UDF instance.
CAUSED BY: InvocationTargetException: null
CAUSED BY: NullPointerException: null</code></pre>

The <code class="language-sql">UDFXPathString</code> class does not violate any of the restrictions dictated by Impala [here](http://www.cloudera.com/content/cloudera/en/documentation/cloudera-impala/latest/topics/impala_udf.html#udfs_hive_unique_2) as it only returns the first result, not an array. I will look into it further, when I get the time, as it could simply be a pesky permissions error, those guys cause all kinds of issues in the Hadoop world.

Summary
------------------------------
In conclusion, XML is _OK_ in Hive but quite restricted without writing a lot of scripts to create tables or custom UDF's to handle it.

####References

- [Cloudera Impala UDFs](http://www.cloudera.com/content/cloudera/en/documentation/cloudera-impala/latest/topics/impala_udf.html#udfs_hive_unique_2)
- [Hive XML SerDe](https://github.com/dvasilen/Hive-XML-SerDe/wiki/XML-data-sources)
- [Hive XPath UDF](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+XPathUDF)
