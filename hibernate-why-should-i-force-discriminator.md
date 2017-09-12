# Hibernate: Why should I Force Discriminator?

[From](http://www.gmarwaha.com/blog/2009/08/26/hibernate-why-should-i-force-discriminator/)  

Hibernate is an ambitious project that aims to be a complete solution to the problem of managing persistent data in java. Even with such an arduous task before them, the hibernate team tries very hard to expose a simple API for developers like us. Still, the complexity behind the API shows its ugly face time and again and I believe it is unavoidable as long as the mismatch between Object and Relational world exists. 

Hibernate 是一个ambitious项目，目标是做一个Java里的管理持久数据的完整解决方案。在Hibernate之前如此费力的任务，hibernate非常努力的尝试来暴露简单的API给开发者，比如我们。尽管如此，API后边复杂的东西一次又一次的展现它的 ugly face time. 我相信这是不可避免的，只要对象和关系的世界存在不匹配之处。  

That said, although I have worked with hibernate for many years and have been its advocate in all my organizations, I keep facing newer issues and keep finding newer ways to work with it efficiently and effectively. Recently, when I was working for nboomi.com, I faced an issue when mapping a OneToMany relationship to the sub-classes of “Single Table Inheritance” strategy. After a frustrating couple of hours of debugging I finally landed on the correct solution. So, I thought other developers who will travel this path could get benefited and started writing this blog post.  

上边的意思是，虽然我用Hibernate很多年并且做在我的组织里做Hibernate的倡导者，我留意Hibernate的新的热点，一直寻找新的方法来高效地有效地使用它。 最近，当我为nboomi.com开发时，我遇到了一个问题，当映射一个OneToMany关系到一个“Single Table Inheritance” strategy的子类时。经过令人沮丧的几个小时的调试后，最后找到了正确的解决方案。 因此，我认为其他的开发者要通过这条路的，可能会有所帮助，所以，我开始写这篇博客。  

Let me explain the issue I faced with an example. Assume you have a normal User Entity with the typical id, version and loginId properties. Assume this User can have many AboutUs sections and many Service sections. You don’t need to be an architect to model them as OneToMany relationships from User. So, I modelled UserAboutSection and UserServiceSection entities and created a OneToMany relationship between User and these entities. Looking at the commonality between these two, I decided to factor out the common fields into a superclass called UserSection. Now, both UserAboutSection and UserServiceSection extends UserSection. I chose to map this Inheritance hierarchy using “Single Table Inheritance” strategy to keep it simple and since most of the fields were common and only a few were specific.  

我用一个例子来解释一下我遇到的这个问题，假设你有一个普通的User Entity有特有的id,version,loginId的属性。假设这个User可以有很多AboutUs sections，很多Service sections.你不需要是一个架构师来给它们和User的一对多关系建模。 所以，我创建了UserAboutSection，UserServiceSection实体，并且创建了OneToMany关系在User和这些实体之间。看到这两个实体之间的共性，我决定重构提取出相同的field到一个superclass称为UserSection.现在，UserAboutSection and UserServiceSection都继承自UserSection.我选择用“Single Table Inheritance” strategy来映射这个层级，来让它保持简单，既然大多数 的field是相同的，只有几个是特殊的。  

The User entity is given below. Notice the List<UserAboutSection> and a List<UserServiceSection> mapped using OneToMany relationship.  

User 实体在下边给出了，注意List<UserAboutSection> and a List<UserServiceSection>使用OneToMany映射。  

The getters, setters, adders, imports and static imports are omitted for brevity.

```   
@Entity @Table(name = "user")
public class User extends BaseEntity {

    @Column(name = "login_id")
    private String loginId;

    @Column(name = "password")
    private String password;

    ...

    @OneToMany(cascade = {CascadeType.ALL})
    @JoinColumn(name = "user_id", nullable = false)
    @IndexColumn(name = "user_section_position")
    List<UserAboutSection> aboutUs;

    @OneToMany(cascade = {CascadeType.ALL})
    @JoinColumn(name = "user_id", nullable = false)
    @IndexColumn(name = "user_section_position")
    List<UserServiceSection> services;

    ... Getters, Setters and Adders
}

```

Here goes the UserSection entity that acts as the base class in this “Single Table Inheritance” strategy. Hibernate uses the @DiscriminatorColumn annotation to distinguish between sub-classes.  

这里是UserSection实体，作为“Single Table Inheritance” 策略的基类，Hibernate使用@DiscriminatorColumn来区别不同的子类。

```
@Entity @Table(name = "user_section")
@Inheritance(strategy= InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name="user_section_type", discriminatorType = STRING) 
public class UserSection extends BaseEntity {
    
    @Column(name = "title")         
    protected String title;

    @Column(name = "description")   
    protected String description;

    ... Getters and Setters
}
```

Here goes the UserAboutSection entity that derives from the UserSection entity. Hibernate uses the @DiscriminatorValue annotation to decide if a row in the database belongs to an instance of this class.

这里是UserAboutSection实体，继承自UserSection实体。Hibernate使用@DiscriminatorValue注解来决定数据库里的一行是否是这个class的实现。

```
@Entity @DiscriminatorValue("ABOUT")
public class UserAboutSection extends UserSection {
    @ManyToOne 
    @JoinColumn(name="user_id",updatable=false,insertable=false,nullable=false)
    protected User user;

    ... Other Properties specific to UserAboutSection
}
```

Here goes the UserServiceSection entity that derives from the UserSection entity. Hibernate uses the @DiscriminatorValue annotation to decide if a row in the database belongs to an instance of this class.
 。。。

```
@Entity @DiscriminatorValue("SERVICE")
public class UserServiceSection extends UserSection {
    @ManyToOne 
    @JoinColumn(name="user_id",updatable=false,insertable=false,nullable=false)
    protected User user;

    ... Other Properties specific to UserServiceSection
}
```

Pretty straightforward… huh! When you try to retrieve an instance of User along with its aboutUs and services collections eagerly (or lazily – doesn’t matter), what do you expect?  

非常直截了当，huh!当你尝试获取一个User实例和他的aboutUs,services集合的时候，eagerly lazily不重要，你期待的是什么？  

I expected an instance of User with the aboutUs collection filled with only UserAboutSection instances and the services collection filled with only UserServiceSection instances corresponding to only the rows they represent in the database. And I believe this expectation is valid, because that is what the mapping looks like and hibernate also has all the information it needs to make this work. 

我期待一个User实例和aboutUs集合，只包含了UserAboutSection的实例，UserServiceSection只包含了UserServiceSection的实例，对应的是数据库了它们代表的行。并且我相信这个预期是合理的，因为这是映射看起来的样子，并且Hibernate有它所需要的所有信息。 

But I got something different. Both the aboutUs and services collections had all the UserSection rows that belong to this User. I mean, aboutUs collection had all the UserSection instances including UserAboutSection and UserServiceSection instances. This was surprising because hibernate has all the information it needs to populate the right instances.  

但是结果还是不一样。aboutUs，services集合包含了这个User所有的UserSection行。我的意思是，aboutUs集合有所有UserSection实例，包括UserAboutSection and UserServiceSection。这是令人惊讶的，因为Hibernate有了所有它需要的信息来移植正确的实例。

After quite a bit of debugging, googling and RTFM-ing I landed upon @ForceDiscriminator annotation. This annotation has to be applied to the base class in the Inheritance hierarchy for “Single Table Inheritance” strategy. In my case, I had to apply it to UserSection entity. The UserSection entity after applying this annotation is given below…  

一番调试，google,RTFM-ing 我找到了@ForceDiscriminator注解。这个注解必须应用在 “Single Table Inheritance”层级的基类上。在我的例子中，我必须把它用到UserSection实体上。加上这个注解之后的UserSection是这个样子。。。

```
@Entity @Table(name = "user_section")
@Inheritance(strategy= InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name="user_section_type", discriminatorType = STRING) 
@ForceDiscriminator
public class UserSection extends BaseEntity {
    
    @Column(name = "title")         
    protected String title;

    @Column(name = "description")   
    protected String description;

    ... Getters and Setters
}
```

Once I ask hibernate to Force Discriminiator, it is happy and populates the aboutUs and services collections with its respective instances.  

一旦我让Hibernate Force Discriminiator，Hibernate成功的填充了aboutUs和services集合，和它各自的实例。  

Ok, Problem Solved! But why did I have to tell hibernate to Force Discriminator. Shouldn’t that be the default behaviour. Is it a bug in hibernate or is it a feature? Am I missing something? If any one of you hibernate fans have walked this path and know the answer, please feel free to drop in a comment. I sincerely hope this post will be a valuable time-saver for other hibernate developers who step on this Bug/Feature.  

Ok,问题解决了，但是为什么我还有必须告诉Hibernate Force Discriminator. 这是Hibernate的一个bug吗，还是它的一个功能。我错过了一些东西吗？如果你们中有Hibernate fans走过这条路，知道这个答案，请随便写点评论。我真诚地希望这篇博客将会是一个有价值的节省时间的东西对其他遇到问题的Hibernate开发者来说。

