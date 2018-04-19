---
title: Hibernate学习笔记（易）
date: 2017-05-08 20:29:16
tags:
- Hibernate
categories:
- 数据库
---

* Hibernate API

    Hibernate API对 JDBC API 进行封装，提供面向对象的数据库访问API，这样不懂数据库语言的人，也能通过Hibernate的接口来访问数据库，无需书写sql语句即可实现对数据的查询操作。增强了代码的独立性和可维护性。

* 对象-关系映射元数据

    Hibernate 将对象-关系映射元素放到对象-关系映射文件中，以hbm.xml命名作为文件扩展名。 下面是monkey类和MONKEYS表的映射关系


    ​```
    
    //指定类和表的映射
    
      //主键
       
       //指定对象标识符生成器，为对象OID生成唯一标识符
         ​
       
      //属性和表的字段映射


​        
​    
    ​```

*   Hierbnate的配置文件
*   Hierbnate的配置文件
    1.XML格式，同上。小tips：可通过直接将数据库中的表与java中对应的实体bean类进行映射,如： 


    ​```
    
    ​```


​    
    2.hibernate.properties：java属性文件，采用“健=值”形式 


​    
    ​```
    //指定数据库使用的sql方言
    hibernate.dialect=org.hibernate.dialect.MYSQLDialect
    //指定数据库驱动程序
    hibernate.connoction.dirver_class=com.mysql.jdbc.Drivcer
    //指定连接数据库的URL
    hibernate.connection.url=jdbc:mysql://localhost:3306/sampleDB
    //指定连接数据库的用户名、口令
    hibernate.connection.username=root
    hibernate.connettion.password=123456
    //值为true时，控制台输出sql语句
    hibernate.show_sql=true
    ​```

*   Hibernate API中的接口
*   Hibernate API中的接口
    1.Configuration接口：配置并启动Hibernate 


    ​```
    //创建一个Configuration类实例，读入hibernate.properties配置文件中的配置信息
    Configuration config = new Configuration();  
    config.addClass(Monkey.class);
    //创建一个sessionfactory实例，把配置信息放入sessionFactory缓存
    sessionFactory=config.buildSessionFactory();  
    ​```


​    
    也可以写成：


​    
    ​```
    seessionFactory = new Configuration().addClass(Monkey.class).buildSessionFactory();     
    ​```


​    
    2.SessionFactory接口：一个SessionFactory实例对应一个数据存储源
    3.Session接口：提供了和持久化相关的操作，如保存、更新、删除、加载对象,可通过SessionFactory实例的openSession()方法获取Session实例。
    Session提供了save()方法，update()方法，delete()方法，get()/load()方法


​    
    ​```
    Session session = factory.openSession(); 
    tx=session.beginTransaction();
    session.save(monkey);
    tx.commit();
    ​```


​    
    4.Transaction接口：是hibernate的数据库事物接口 


​    
    ​```
    Transaction tx;
        try{
            tx=session.beginTransaction();
            //执行事务；
            ..
            //提交事务
            tx.commit();
        }catch(RuntimeException e){
            if(tx!=null) tx.rollback();
            throw e;
        }finally{
            session.close();
        }
    ​```


​    
    5.Query接口：Hibernate的查询接口,


​    
    ​```
    Query query=session.createQuery("from Monkey as m order by m.name by m.name asc");
    ​```

*   OID
*   OID
    Hibernate通过OID来维持java对象和数据库表中的对应关系。Hibernate会保证在一个Session对象的缓存中，每个Monkey对象都有唯一的OID（提取两个Monkey1，Monkey2对应的是同一个对象）


    ​```
    Monkey monkey1 =(Monkey)session.get(Monkey.class,new Long(1));
    Monkey monkey2 =(Monkey)session.get(Monkey.class,new Long(1));
    ​```

*   Session缓存的作用
*   Session缓存的作用
    Session缓存可以减少访问数据库的频率；
    Session缓存中保证数据库中的相关记录与缓存中的相应对象保持同步。

*   Hibernate的检索方式
    1.HQL检索（hibernate Query Language）
    与sql相似，通过Session的createQuery方法创建一个Query对象，然后动态绑定参数，通过list方法执行查询语句。


    ​```
    Query query=session.createQuery("from Monkey as m where m.name=:monkeyName "+"and m.age=:monkeyAge")
    
    query.setString("MonkeyName","Tom");
    query.setInteger("monkeyAge",21);
    
    List result=query.list();
    ​```


​    
    或者采用方法链编程风格：


​    
    ​```
    Query query=session.createQuery("from Monkey as m where m.name=:monkeyName "+"and m.age=:monkeyAge")
    .setString("MonkeyName","Tom")
    .setInteger("monkeyAge",21)
    .list();
    ​```


​    
    2.QBC检索方式 QBC API由org.hibernate.Criteria接口和org.hibernate.criterion.Criterion接口和org.hibernate.criterion.Restrictions类组成


​    
    ​```
    List result=session.createCriteria(Monkey.class)
    .add(Restrictions.like("name","T%"))
    .add(Restrictions.eq("age",newInteger(21)))
    .list();
    ​```


​    
    3.sql检索方式 有的时候可能需要根据底层数据库的sql方言来生成一些特殊的查询语句，这时候将sql语句代入即可


​    
    ​```
    Query query=session.createQuery("select * from MONKEYS where NAME like :monkeyName "+"and AGE=:monkeyAge")
    .setString("MonkeyName","Tom")
    .setInteger("monkeyAge",21)
    .list();
    ​```

*   实际使用
*   实际使用
    在实际使用时，一般会根据自己的需求情况对Hibernate进行封装，通过将Hibernate类封装形成自己的Dao父类，实现ListBySQL(String sql,Class c),ListByCriteria(DetachedCriteria criteria)等方法，将会在自己写接口，调用数据库时事半功倍！只用考虑接口如何实现而不需在繁琐的建立JDBC连接等等一系列操作啦~~非常方便快捷~