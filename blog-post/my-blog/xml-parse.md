---
tags: performance, xml, benchmark
---

In the early era of internet XML was the hot topic. It is widely used in webservices and systems that need to communicate with each other. XML is providing structured approach on communications that’s why there are some specifications built on that. For example SOAP,XBRL,UBL. 

In this post we will discover how can you parse XML files in .NET and what are the speed of this methods. 

For benchmarking purposes we will use glorious Benchmark.NET. It is widely adopted and powerful library for benchmarking both your application and code blocks that you want measure performance. Generally this library is used for code blocks which is responsible for one operation and since our topic is XML parsing, we will use code blocks only for parsing XML files. 

As for the input and test values we will be using some XML files which is consist of small, medium and large size which you should be able to identify from the results.

So lets start learning how to parse XML in .NET and how much fast each method is. 

In software development there will be always choices that you need to make to built things for your customers. These choices can depend on company culture, customer satisfaction etc. There are no methods that we will find out is silver bullet for parsing but there will be different approaches for different kind of needs. 


## XmlReader

``XmlReader`` is a class for reading, parsing and forward only iterating on XML documents. What is forward only means that you will be reading XML document top to bottom with loops and not able to read previous nodes on XML files. 

This class is the best when you have big XML file, need a performance, know the structure of XML document and sure about it will not change its structure. Since it is parsing document on iteration it will not consume that much memory and you will be able to only parse for what you are looking for. 

These limitations can create a lot of code when parsing document but performance comes with downside as well.
 

> ### ⚠ Security considerations
> ``XmlReader`` class have an option class (``XmlReaderOptions``)  when parsing document. You should look for this options when reading XML files in order to prevent XXE vulnerability 
> More information can be found on [here](https://docs.microsoft.com/en-us/dotnet/api/system.xml.xmlreader?view=net-6.0#security-considerations=)

You can find basic usage of this class below.

````csharp
using (var reader = XmlReader("xmlfile.xml"))
{
    while(reader.Read())
    {
        switch (reader.NodeType)
        {
            case XmlNodeType.Element:
                Console.WriteLine("Start Element {0}", reader.Name);
                break;
            case XmlNodeType.Text:
                Console.WriteLine("Text Node: {0}",
                            await reader.GetValueAsync());
                break;
            case XmlNodeType.EndElement:
                Console.WriteLine("End Element {0}", reader.Name);
                break;
            default:
                Console.WriteLine("Other node {0} with value {1}",
                                reader.NodeType, reader.Value);
                break;
        }
    }
}
````
## XDocument

.NET is providing a black magic level of technology which is called LINQ. It provides human readable stream processing interfaces and methods which provides a lot of flexible way of processing data in streams.

``XDocument`` is providing LINQ API to use in XML documents. Which can be very powerful when reading/parsing if your XML file is deep on nodes and their parents itself. Even query XML is process with stream first it needs to be loaded on memory. This can create memory issues in your application if your source XML file is big. If you need to parse small XML files with much more developer friendly way you should consider using ``XDocument``.

XDocument class also provides XPath extensions to query nodes with XPath expressions. But for working with XPath you should have prior knowledge about XML Namespaces used in your XML files

You can find below basics of reading and query data with ``XDocument``


````csharp
XDocument document = XDocument.Load("xmlfile.xml");

//Select every SomeChidNode belongs to SomeNode
document.Descendants("SomeNode").Select(x => x.Name.Equals("SomeChildNode"));

//Much more complex queries
document.Descendants()
    .Where(x => x.Name.Equals("SomeValue") && x.Value == 5)
    .Select(x => new SomeCustomClass{ Property1 = x.Value, Property2 = x.Name});
---
XmlNamespaceManager namespaceManager = new XmlNamespaceManager(document.NameTable);
namespaceManager.AddNamespace("myns", "mynamespacename");
document.XPathSelectElements("//some/xpath/query", namespaceManager);

````

## String Operations

This method is a custom approach and not topic of this post. Also, it is much more specific for your data. 

In .NET you have rich string operations when manipulating and reading strings. For the XML parsing part, if you %100 sure about your XML data structure and it will never change based on different input you can elevate these string operations to process your XML files.

One example that come to my mind when parsing XML document with string operations which can be found below. There can be different implementations for every data. I am not fan of manual string operations when working with structured data but it is good to know there is also this way to do.


````csharp
string xmlDocument = File.ReadAllText("xmlFile.xml");
string data = string.Substring(xmlDocument.IndexOf("<SomeNode>"),xmlDocument.LastIndexOf("</SomeNode>"))
````

## Benchmarking

So far, we have discovered different methods when parsing XML files on .NET let's measure their performance now. 
We will be using one small and one large XML file for investigating performance.

Small XML file will be Invoice XML which defined in UBL standard itself. And we will be reading value of UUID tag inside of it. 


````xml
<Invoice xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2" xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2" xmlns:xades="http://uri.etsi.org/01903/v1.3.2#" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:ext="urn:oasis:names:specification:ubl:schema:xsd:CommonExtensionComponents-2" xmlns:ds="http://www.w3.org/2000/09/xmldsig#" xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2">
	...
	<cbc:UUID>0eb479ed-9a11-4577-95de-6a8879002e16</cbc:UUID>
	...
</Invoice>
````

Big XML file will be XBRL XML which defined in XBRL standard. This file used mainly on accounting purposes and used for digital ledger of company itself. We will be reading last record of this document for the see differences in benchmarking. While this file has no limit on keeping record, we will be keeping file size as 50MB. 

````xml
<xbrl>
...
    <gl-cor:entryHeader>
        <gl-cor:entryNumber contextRef="journal_context">154687</gl-cor:entryNumber>
        <gl-cor:entryNumberCounter contextRef="journal_context" decimals="INF" unitRef="countable">
        154687
        </gl-cor:entryNumberCounter>
        ...
    </gl-cor:entryHeader>
...
</xbrl>
````
### Benchmark Codes
---
````csharp
[Benchmark]
public void XDocumentSmallXml()
{
    XmlNamespaceManager xmlNamespaceManager = new XmlNamespaceManager(new NameTable());
    xmlNamespaceManager.AddNamespace("cbc", "urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2");

    XDocument.Parse(File.ReadAllText(smallXml)).XPathSelectElements("//cbc:UUID", xmlNamespaceManager).Select(x => x.Value).ToList();
}

[Benchmark]
public void XmlReaderSmallXml()
{
    using (var reader = XmlReader.Create(smallXml))
    {
        while (reader.Read())
        {
            if (reader.NodeType == XmlNodeType.Element && reader.LocalName == "UUID")
            {
                reader.ReadElementContentAsString();
            }
        }
    }
}

[Benchmark]
public void XmlReaderBigXml()
{
    int value = 0;
    using (var reader = XmlReader.Create(bigXml))
    {
        while (reader.Read())
        {
            if (reader.NodeType == XmlNodeType.Element && reader.LocalName == "entryNumberCounter")
            {
                int tempvalue = reader.ReadElementContentAsInt();
                if (value < tempvalue) value = tempvalue;
            }
        }
    }
}
[Benchmark]
public void XDocumentBigXml()
{
    XmlNamespaceManager xmlNamespaceManager = new XmlNamespaceManager(new NameTable());
    xmlNamespaceManager.AddNamespace("gl-cor", "http://www.xbrl.org/int/gl/cor/2006-10-25");

    XDocument.Parse(File.ReadAllText(bigXml)).XPathSelectElements("//gl-cor:entryNumber", xmlNamespaceManager).Max(x => Convert.ToInt32(x.Value));
}

````
### Results
---


|            Method |         Mean |        Error |       StdDev |      Gen 0 |      Gen 1 |     Gen 2 |    Allocated |
|------------------ |-------------:|-------------:|-------------:|-----------:|-----------:|----------:|-------------:|
| XDocumentSmallXml |   1,048.7 us |     12.90 us |     11.43 us |   332.0313 |   332.0313 |  332.0313 |   1937.26 KB |
| XmlReaderSmallXml |     437.2 us |      2.63 us |      2.46 us |     7.8125 |     0.4883 |         - |     37.55 KB |
|   XmlReaderBigXml | 163,757.6 us |  2,802.65 us |  2,484.48 us |          - |          - |         - |    608.04 KB |
|   XDocumentBigXml | 923,778.1 us | 17,583.96 us | 16,448.05 us | 53000.0000 | 21000.0000 | 6000.0000 | 397730.95 KB |

As you can see from results ``XmlReader`` class go to if you are reading/parsing big files. However, there are many points that can be improved in benchmark code. For the ``XDocument`` instead of using ``Max`` method we could use aggregate method built in LINQ. I will leave that to you because we are learning together. 

For the small sized XML files if the performance is not must for you can use ``XDocument`` class since it is more developer friendly. If you are working on a project which many teams involved, you can choose this method. 

## Conclusion
We investigated different XML parsing methods available in .NET and looked their performance in different circumstances. As I mentioned there is no silver bullet for parsing but different approaches for different needs. 

Feel free to reach me out if you have any questions about this topic. Also let me know if you have another comparisons for XML parsing as well. 


## References
XmlReader : https://docs.microsoft.com/en-us/dotnet/api/system.xml.xmlreader?view=net-6.0#examples=

XDocument : https://docs.microsoft.com/en-us/dotnet/api/system.xml.linq.xdocument?view=net-6.0