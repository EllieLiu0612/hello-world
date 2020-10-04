# pj1：实现一个简单的REST风格单体Web应用
姓名：刘香君<br>
学号：20212010053

### 任务描述
选择语言和框架，实现一个简单的REST风格单体Web应用。<br>
编写Dockerfile，使用docker将服务封装为镜像，要求通过镜像可在本机访问到相应API。

### 功能需求
对学生信息进行管理，学生实体中包括字段：学号，姓名，院系，专业。<br>
该服务要实现对学生的增删改查功能。
### 项目实现
在此实践中我选择了java语言，使用spring boot框架。<br><br>
1.Student实体定义<br>
```
import java.io.Serializable;

public class Student implements Serializable{
	private Long studentId;
	private String name;
	private String department;
	private String major;
	Student(){}
	public Student(Long studentId, String name, String department, String major) {
		super();
		this.studentId = studentId;
		this.name = name;
		this.department = department;
		this.major = major;
	}
	public Long getStudentId() {
		return studentId;
	}
	public void setStudentId(Long studentId) {
		this.studentId = studentId;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getDepartment() {
		return department;
	}
	public void setDepartment(String department) {
		this.department = department;
	}
	public String getMajor() {
		return major;
	}
	public void setMajor(String major) {
		this.major = major;
	}
}
```

2. 由于实践需求不需要前端和底层数据库实现，故将数据封装在数据结构中，只需要一个controller类即可实现功能。<br>

```
@RestController
@RequestMapping(value="/api/v1/student")
public class StudentController { 

    static Map<Long, Student> students = Collections.synchronizedMap(new HashMap<Long, Student>());

    @RequestMapping("/hello")
    public String hello() {
        Student s1=new Student((long) 1,"liuxiangjun","software","machine learning");
        Student s2=new Student((long) 2,"dingmeifei","software","machine learning");
        Student s3=new Student((long) 3,"zangxuan","software","machine learning");

        students.put((long) 2,s2);
        students.put((long) 3,s3);
        students.put((long) 1,s1);
        return "hello world! Initialization completed";
    }

    @RequestMapping(method=RequestMethod.POST)
    public List<Student> postUser(@ModelAttribute Student s) {
    	students.put(s.getStudentId(), s);
        System.out.println("add success");
        List<Student> slist = new ArrayList<Student>(students.values());
        return slist;
    }

    @RequestMapping(method=RequestMethod.GET)
    public List<Student> getStudentList() {
        List<Student> slist = new ArrayList<Student>(students.values());
        System.out.println("get success");
        return slist;
    }


    @RequestMapping(method=RequestMethod.PUT)
    public List<Student> putUser(@ModelAttribute Student student) {
        Long id=student.getStudentId();
        Student u = students.get(id);
        u.setName(student.getName());
        u.setDepartment(student.getDepartment());
        u.setMajor(student.getMajor());
        students.put(id, u);
        System.out.println("update studentId:"+id+" success");
        List<Student> slist = new ArrayList<Student>(students.values());
        return slist;
    }

    @RequestMapping(method=RequestMethod.DELETE)
    public List<Student> deleteStudent(@ModelAttribute Student student) {
        Long studentId=student.getStudentId();
        students.remove(studentId);
        System.out.println("delete success");
        List<Student> slist = new ArrayList<Student>(students.values());
        return slist;
    }
}
```
3. 为了查看功能是否实现好，增加测试类，用于单元测试。可直接运行测试四个方法的功能。

```
//import部分略去
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = MockServletContext.class)
@WebAppConfiguration
public class Boot1ApplicationTests {

	private MockMvc mvc;

	@Before
	public void setUp() throws Exception {
		mvc = MockMvcBuilders.standaloneSetup(new StudentController()).build();
	}

	@Test
	public void testUserController() throws Exception {
		RequestBuilder request = null;
		// 1、post提交一个student
		request = post("/api/v1/student")
				.param("studentId", "4")
				.param("name", "xieming")
				.param("department", "computer science")
				.param("major", "data science");
		mvc.perform(request)
				.andExpect(content().string(equalTo("[{\"studentId\":4,"
						+ "\"name\":\"xieming\","
						+ "\"department\":\"computer science\","
						+ "\"major\":\"data science\"}]")));


		// 2、get获取student列表
		request = get("/api/v1/student");
		mvc.perform(request)
				.andExpect(status().isOk())
				.andExpect(content().string(equalTo("[{\"studentId\":4,"
						+ "\"name\":\"xieming\","
						+ "\"department\":\"computer science\","
						+ "\"major\":\"data science\"}]")));


		// 3、put修改id为4的student
		request = put("/api/v1/student")
				.param("name", "zangxuan")
				.param("department", "computer science")
				.param("major", "data science")
				.param("studentId", "4");
		mvc.perform(request)
				.andExpect(content().string(equalTo("[{\"studentId\":4,"
						+ "\"name\":\"zangxuan\","
						+ "\"department\":\"computer science\","
						+ "\"major\":\"data science\"}]")));
		// 4、del删除id为4的student
		request = delete("/api/v1/student")
				.param("studentId", "4");
		mvc.perform(request)
				.andExpect(content().string("[]"));
	}
```

4. 如用浏览器访问
* 启动Boot1Applcation类，该类为spring boot框架自动生成，即可用浏览器访问
* 启动Boot1ApplicationTests类，可进行单元测试
* map用于存储学生信息，如在浏览器访问，可先访问http://localhost:8080/api/v1/student/hello 用于初始化数据化数据
* 访问http://localhost:8080/api/v1/student ，可查看初始化后的学生信息
* 删除，修改，以及增加学生，由于没有前端实现，可借助postman工具实现。

5. 关于docker镜像<br>
我将docker安装在了虚拟机中，设置ip和端口号为：192.168.100.3:8000。<br>
封装好镜像后，为镜像设置一个docker容器，开启容器后即可访问，借助postman访问http://192.168.100.3:8000/api/v1/student 即可<br>
![1.init](img/31.png)<br>
![2.getall](img/32.png)<br>
![3.add](img/33.png)<br>
![4.update](img/34.png)<br>
![5.delete](img/35.png)<br>

### 实践总结
* rest风格可以便捷开发。资源通过 URL 进行识别和定位，然后通过行为(即 HTTP 方法)来定义 REST 来完成怎样的功能。
* docker可以打包项目和包到镜像中，并通过建立容器，实现快捷交付，在微服务开发中非常有意义。
* 通过这个项目，简单实现了springboot小项目，实现了增删改查的基本功能，期待之后加入前端和数据库，让项目更加完整。
