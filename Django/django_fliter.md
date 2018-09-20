### 准备工作 ###
#### 1.mysql数据库中创建定义的数据库

a) 进入mysql

    mysql -u root -p
    密码

b) 创建数据库

    create database 数据库名 default charset urf8;

#### 2.在settings.py中配置数据库连接信息

    'ENGINE':'django.db.backends.mysql',
    'NAME': 'srs',           #数据库名
    'USER': 'root',        #账号
    'PASSWORD': '123456',    #密码
    'HOST':'127.0.0.1',           #IP(本地地址也可以是localhost)
    'PORT':'3306',     #端口

#### 3.在settings.py中将应用加到INSTALLED_APPS中
    
    INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'hello'    # 应用名称
]

#### 4.在工程目录day01/\_\_init__.py中输入如下代码，完成数据库的驱动加载
   
     import pymysql


    pymysql.install_as_MySQLdb()

#### 5.定义模型

<h6>重要概念：模型，表，属性，字段</h6>

一个模型类在数据库中对应一张表，在模型类中定义的属性，对应模型对照表中的一个字段

定义属性见定义属性文件地址

<h6>创建学生模型类</h6>

 在hello/models.py中加入如下代码：

  <pre>
 class Student(models.Model):

    stu_name = models.CharField(max_length=10, unique=True)
    stu_age = models.IntegerField(default=16)
    stu_sex = models.BooleanField(default=1)
    Chinese = models.DecimalField(max_digits=5, decimal_places=2, null=True)
    maths = models.DecimalField(max_digits=5, decimal_places=2, null=True)

    class Meta:
        db_table = 'student'
</pre>

#### 6.迁移数据库

a) 生成迁移文件

    python manage.py makemigrations

> 注意：如果执行后并没有生成迁移文件，一直提示No changes detected这个结果的话，就要手动的去处理了。有两点处理方式：

 　1） 先删除掉__pycache__文件夹

　 2） 直接强制的去执行迁移命令，python manage.py makemigrations xxx (xxx就是app的名称)

　 3） 查看自动生成的数据库，查看表django_migrations，删掉app字段为xxx的数据(xxx就是app的名称)

b) 执行迁移生成数据库

    python manage.py migrate

> 注意: 生成迁移文件的时候，并没有在数据库中生成对应的表，而是执行migrate命令之后才会在数据库中生成表

a. 设置路由，hello/urls.py中代码如下：

<pre>
urlpatterns = [
    # 127.0.0.1:8000/app/helloWorld/
    url(r'helloWorld/', views.hello),
]
</pre>
b. hello/views.py中代码如下：
<pre>
def hello(request):
    if request.method == 'GET':
        return HttpResponse('hello World')
</pre>

#### 7.模型的CRUD操作

<i><b>ORM</i></b>

ORM(Objects Relational Mapping)对象关系映射，是一种程序技术，用于实现面向对象编程语言里不同类型系统的数据之间的转换。可以简单理解为翻译机。

##### 查询(R)

[官网网址：https://docs.djangoproject.com/zh-hans/2.0/ref/models/querysets/](https://docs.djangoproject.com/zh-hans/2.0/ref/models/querysets/)

<b>1. 查询所有学生信息</b>
<pre>
<b>a. hello/urls.py中新增代码</b>
urlpatterns = [
    # 127.0.0.1:8000/app/helloWorld/
    url(r'helloWorld/', views.hello),

    <font color='green'># 新增代码</font>
    # 查询所有的学生信息
    url(r'selStudent/', views.selStu),
]

<b>b. hello/views.py中新增代码如下：</b>

def selStu(request):
    if request.method == 'GET':
        stus = Student.objects.all()   # 得到的是QuerySet,需要遍历才能得到学生实例
   
        stus_list = []   # 用列表来存储这些实例
        for stu in stus:
            s = {
                'stu_name': stu.stu_name,
                'stu_age': stu.stu_age
            }
        stus_list.append(s)    # 以字典的格式将实例添加到列表中
        return HttpResponse(stus)

或者：
1. 在models.py中加入如下代码：

    def to_dict(self):
        return {
            'id': self.id,
            'stu_name': self.stu_name,
            'stu_age': self.stu_age,
            'stu_sex': self.stu_sex,
            'Chinese': self.Chinese,
            'maths': self.maths
        }
2. hello/views.py中新增代码如下：
def selStu(request):
    if request.method == 'GET':
        stus = Student.objects.all()   # 得到的是QuerySet,需要遍历才能得到学生实例
       # 列表推导式 [i for i in seq]
       stus_list = [stu.to_dict() for stu in stus]
       return HttpResponse(stus_list)
       # return HttpResponse(stus)
</pre>

<b>2.根据条件查询</b>

　数据库API截取的一部分：
<table>
		<tr>
			<th>类型</th>
			<th>描述</th>
		</tr>
		<tr>
			<td>exact</td>
			<td>精确匹配: polls.get_object(id__exact=14).</td>
		</tr>
		<tr>
			<td>iexact</td>
			<td>忽略大小写的精确匹配: polls.objects.filter(slug__iexact="foo") 匹配 foo, FOO, fOo, 等等.</td>
		</tr>
		<tr>
			<td>contains</td>
			<td>大小写敏感的内容包含测试: polls.objects.filter(question__contains="spam") 返回question 中包含 "spam" 的所有民意测验.(仅PostgreSQL 和 MySQL支持. SQLite 的LIKE 语句不支持大小写敏感特性. 对Sqlite 来说, contains 等于 icontains.)</td>
		</tr>
		<tr>
			<td>icontains</td>
			<td>大小写不敏感的内容包含测试:</td>
		</tr>
		<tr>
			<td>startswith</td>
			<td>大小写敏感的 starts-with: polls.objects.filter(question__startswith="Would"). (仅PostgreSQL 和MySQL支持. SQLite 的LIKE 语句不支持大小写敏感特性. 对Sqlite 来说,``startswith`` 等于 istartswith)</td>
		</tr>
		<tr>
			<td>endswith</td>
			<td>大小写敏感的 ends-with. (仅PostgreSQL 和 MySQL)</td>
		</tr>
		<tr>
			<td>istartswith</td>
			<td>大小写不敏感的 starts-with.</td>
		</tr>
		<tr>
			<td>iendswith</td>
			<td>大小写不敏感的 ends-with.</td>
		</tr>
		<tr>
			<td>range</td>
			<td>范围测试: polls.objects.filter(pub_date__range=(start_date, end_date)) 返回 pub_date 位于 start_date 和 end_date (包括)之间的所有民意测验</td>
		</tr>
		<tr>
			<td>year</td>
			<td>对 date/datetime 字段, 进行精确的 年 匹配: polls.get_count(pub_date__year=2005).</td>
		</tr>
		<tr>
			<td>month</td>
			<td>对 date/datetime 字段, 进行精确的 月 匹配</td>
		</tr>
		<tr>
			<td>day</td>
			<td>对 date/datetime 字段, 进行精确的 日 匹配</td>
		</tr>
		<tr>
			<td>isnull</td>
			<td>True/False; 做 IF NULL/IF NOT NULL 查询: polls.objects.filter(expire_date__isnull=True).</td>
		</tr>
	</table>

例如：
<pre>
<b>a. hello/urls.py文件，继续向urlpatterns = []列表中新增代码</b>

    # 按条件筛选学生信息
    url(r'filterStudent/', views.filterStu),

<b>b.hello/views.py 中新增代码</b>

def filterStu(request):
    if request.method == 'GET':
	<font color='deepgreen' ># 查询年龄=18的学生, filter()</font>
	stus = Student.objects.filter(stu_age=18)
	
	<font color='deepgreen'># 查询年龄不等于18的学生，exclude()--不包含</font>
	# stus = Student.objects.exclude(stu_age=18)
	
	<font color='deepgreen' ># 排序</font>
	# 按照id升序
	# stus = Student.objects.all().order_by('id')
	# 按照id降序
	# stus = Student.objects.all().order_by('-id')
	
	<font color='deepgreen' ># values()
	- 返回一个QuerySet在用作iterable时返回字典而不是模型实例。
	- 每个字典代表一个对象，其中的键对应于模型对象的属性名称。
	</font>
	# stus = Student.objects.all().values()
	
	<font color='deepgreen' ># get()和filter()</font>
	# 获取id为1的信息
	# stus = Student.objects.filter(id=1)
	# stus = Student.objects.get(id=1)
	# get获取不到数据会报错
	# stus = Student.objects.get(pk=5)
	# filter获取不到数据会返回一个空列表
	# stus = Student.objects.filter(id=5)
	
	# get 只能返回一个数据 ，返回多个会报错
	# stus = Student.objects.get(stu_age=15)
	
	<font color='deepgreen' ># 获取第一条信息first()</font>
	# 获取所有学生（按照id降序）中第一个学生信息
	# stus = Student.objects.all().order_by(-id)[0]
	# stus = Student.objects.all().order_by(-id).first()

	<font color='deepgreen' ># 获取最后一条信息 last()</font>
	# 获取所有学生（按照id降序）中最后一个学生信息
	# stus = Student.objects.all().order_by(-id).last()
	
	<font color='deepgreen'># 模糊查询，包含(__contains)、以...开始(__startswith)、以...结束(__endswith)</font>
	# stus = Student.objects.filter(stu_name__contains='小')
	
	# stus = Student.objects.filter(stu_name__startswith='小')
	# stus = Student.objects.filter(stu_addr__endswith='小')
	
	<font color='deepgreen'># gt:> ,  gte: >= , lt:< , lte:<=</font>
	# stus = Student.objects.filter(stu_age__gt=16)
	# stus = Student.objects.filter(stu_age__gte=16)
	
	<font color='deepgreen'># 位于给定的值中 __in</font>
	# 查询id=1-5的学生信息
	# stus = Student.objects.filter(id__in=[1,  2, 3, 4, 5])

	<font color='deepgreen'># 多个条件查询</font>
	# stus = Student.objects.filter(stu_name__contains='小', stu_age__gt=17)
	
	# 查询语文成绩最高的学生
	# stus = Student.objects.all().order_by('-Chinese').first()
	# stus = Student.objects.all().order_by('Chinese').last()
	
	# 查询比id=1的学生的数学成绩高10分的学生的信息
	# stu = Student.objects.filter(pk=1).first()
	# stu = Student.objects.get(pk=1)
	# maths = stu.maths
	# stus = Student.objects.filter(maths__gt=maths+10)
	
	# 查询数学成绩比语文成绩大10分的学生信息
	<font color='deepgreen'># F()可以将模型的两个字段进行算术计算</font>
	stus = Student.objects.filter(maths__gt=F('Chinese') + 10)
	
	stus = Student.objects.all()
	stus_list = []
	for stu in stus:
	   if stu.maths > stu.Chinese + 10:
	       stus_list.append(stu)
	
	# 查询语文成绩大于70分，或者年龄小于17岁的学生信息
	<font color='deepgreen'># Q()函数</font>
	stus = Student.objects.filter(Chinese__gt=70, stu_age__lt=17)
	stus = Student.objects.filter(Q(Chinese__gt=70) | Q(stu_age__lt=17))
	
	<font color='deepgreen'>~ 非，｜或，&与</font>
	# 查询语文成绩不等于70，或者年龄小于17岁的学生信息  ~ 非，｜或，&与
	stus = Student.objects.filter(~Q(Chinese__gt=70) | Q(stu_age__lt=17))
</pre>

##### 增加(C)
例：添加学生
<pre>
<b>a. hello/urls.py文件，继续向urlpatterns = []列表中新增代码</b>

    # 增加数据
    url(r'addStudent/', views.addStu),

<b>b. hello/views.py文件新增代码</b>
    # 第一种方法
    def addStu(request):
        if request.method == 'GET':
            stu = Student()
            stu.stu_name = '小兰'
            stu.stu_age = '28'
            stu.Chinese = 59
            stu.maths = 30.5
            stu.save()  # 保存新增的实例

    # 第二种 : 重写父类__init__ 方法
    def addStu(request):
        if request.method == 'GET':
            stu = Student('小芳2', '28', 90, 89.6)
            stu.save()
<pre>
需在models.py中加入以下代码
def __init__(self, stu_name, stu_age, Chinese, maths):
    super(Student, self).__init__()
    # super().__init__()
    self.stu_name = stu_name
    self.stu_age = stu_age
    self.Chinese = Chinese
    self.maths = maths
</pre>
    # 第三种方法
    def addStu(request):
        if request.method == 'GET':
	    Student.objects.create(stu_name='妲己', stu_age='40', Chinese=67, maths=56)
	    return HttpResponse('创建成功')
</pre>

##### 删除(D)
<pre>
<b>a. hello/urls.py文件，继续向urlpatterns = []列表中新增代码</b>
    # 删除数据
    url(r'delStudent/', views.delStu),

<b>b. hello/views.py文件新增代码</b>
    # 删除学生
    def delStu(request):
        if request.method == 'GET':
            stu = Student.objects.filter(id=2)
            stu.delete()
            return HttpResponse('删除成功')
</pre>

##### 更新(U)
<pre>
<b>a. hello/urls.py文件，继续向urlpatterns = []列表中新增代码</b>
    # 更新数据
    url(r'updateStudent/', views.updateStu),

<b>b. hello/views.py文件新增代码</b>
    # 更新学生信息
    def updateStu(request):
        if request.method == 'GET':
            # 第一种
            # stu = Student.objects.filter(id=2).first()
            # stu.stu_name = '小山竹'
            # stu.save()

            # 第二种
            Student.objects.filter(id=3).update(stu_name='丁小')
            return HttpResponse('修改成功')
</pre>

#### 7.模型查询

##### a)  模型成员objects 

Django默认通过模型的objects对象实现模型数据查询

##### b) 过滤器

查询集表示从数据库获取的对象集合,查询集可以有多个过滤器,过滤器就是一个函数，基于所给的参数限制查询的结果。

    Django有两种过滤器用于筛选记录

    filter	  : 返回符合筛选条件的数据集

    exclude   : 返回不符合筛选条件的数据集


    当然还有如下这些过滤器:

    all()            返回所有数据

    filter()	         返回符合条件的数据

    exclude()        过滤掉符合条件的数据

    order_by()       排序

    values()         一条数据就是一个字典，返回一个列表

##### c)查询单条数据

	get()：返回一个满足条件的对象。如果没有返回符合条件的对象，会应该模型类DoesNotExist异常，如果找到多个，会引发模型类MultiObjectsReturned异常

	first()：返回查询集中的第一个对象

	last()：返回查询集中的最后一个对象

	count()：返回当前查询集中的对象个数

	exists()：判断查询集中是否有数据，如果有数据返回True，没有返回False

##### d)限制查询集
	
	限制查询集，可以使用下表的方法进行限制，等同于sql中的limit

	模型名.objects.all()[0:5]      #  下标不能为负数