#### &#x1F4DA; [Posts](../)

## XML 映射为 Java 对象

目标：以 XML 文件 [result.mpd](res/result.mpd) 为例，我们希望将其映射为 Java 对象 MPD 以便更好地进行数据处理。

- 从 XML 文件生成相应的 XSD(XML Schema Definition) 文件。
  ```cpp
  XmlReader^ reader = XmlReader::Create("d:\\result.mpd");
  XmlSchemaSet^ schemaSet = gcnew XmlSchemaSet();
  XmlSchemaInference^ inference = gcnew XmlSchemaInference();
  schemaSet = inference->InferSchema(reader);

  FileStream^ file = gcnew FileStream("d:\\result.xsd",
      FileMode::Create, FileAccess::ReadWrite);
  XmlTextWriter^ writer = gcnew XmlTextWriter(file, gcnew UTF8Encoding());

  for each (XmlSchema^ schema in schemaSet->Schemas()) {
    schema->Write(file);
  }
  ```

- 从 XSD 文件生成 Java 类。
  ```bash
  xjc -d "d:\\result" -p chow.dan.mpd "d:\\result.xsd"
  ```

	其中，-d 指定保存目录(directory)，-p 指定类所属包(package)，该命令生成 MPD.java，ObjectFactory.java，package-info.java 三个文件。

- 在 Java 中，通过 JAXB(Java Architecture for XML Binding) 直接解析 XML 文件，可得到 MPD 类的实例。
  ```java
  JAXBContext context = JAXBContext.newInstance(MPD.class);
  Unmarshaller unmarshaller = context.createUnmarshaller();
  MPD mpd = (MPD) unmarshaller.unmarshal(new ByteArrayInputStream(content.getData()));
  ```

### References

- [XmlSchemaInference Class](https://msdn.microsoft.com/en-us/library/system.xml.schema.xmlschemainference(v=vs.110).aspx) 提供了XmlSchemaInference类说明和样例代码。

- [xjc](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/xjc.html) 提供了xjc工具说明。

- [Java Architecture for XML Binding (JAXB)](http://www.oracle.com/technetwork/articles/javase/index-140168.html) 介绍了JXAB及其使用。

#### &#x1F4DA; [Posts](../)