# Content Providers

Content providers关联到结构化数据的访问。它们封装数据，提供定义数据安全的机制。 Content providers是连接两个进程间数据的接口。

要项访问content provider中的数据，你需要应用Context中的ContentResolver对象，作为与provider通信的客户端。ContentResolver与provider（实现ContentProvider类）对象通信。

如果你的应用不需要向其他应用分析数据，则你不需要开发自己的provider。However, you do need your own provider to provide custom search suggestions in your own application. You also need your own provider if you want to copy and paste complex data or files from your application to other applications.

Android自己包含一些content providers，提供audio, video, images, personal contact等数据。参见`android.provider`包。With some restrictions, these providers are accessible to any Android application.

## Content Provider 基础

一个content provider管理到一个数据中央仓库的访问。A provider is part of an Android application, which often provides its own UI for working with the data. However, content providers are primarily intended to be used by other applications, which access the provider using a provider client object. Together, providers and provider clients offer a consistent, standard interface to data that also handles inter-process communication and secure data access.










