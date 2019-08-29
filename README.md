## Requirement

一、mockmvc测试的api
(1)@RunWith注解：指定测试运行器。
 例如使用 SpringJUnit4ClassRunner.class

(2)@ContextConfiguration注解：执行要加载的配置文件。
例如 classpath:application.xml 或 file:src/main/resources/DispatcherServlet-servlet.xml

(3)@WebAppConfiguration注解：用于声明测试时所加载的是WebApplicationContext【WebMVC的 XmlWebApplicationContext 是其实现类】，因为测试需要使用WebMVC对应的IOC容器对象。

⑷ WebApplicationContext：WebMVC的IOC容器对象，需要声明并通过@Autowired自动装配进来

⑸ MockMvcRequestBuilders：用于构建MockHttpServletRequestBuilder
① get GET请求
② post POST请求
③ put PUT请求
④ delete DELETE请求
⑤ param(String name, String… values) 传递参数 K-V…

⑹ MockHttpServletRequestBuilder：用于构建 MockHttpServletRequest，它用于作为 MockMvc的请求对象

⑺ MockMvc：通过 MockMvcBuilders 的 webAppContextSetup(WebApplicationContext context) 方法 获取 DefaultMockMvcBuilder，再调用 build() 方法，初始化 MockMvc。

① perform：perform(RequestBuilder requestBuilder) throws Exception执行请求，需要传入 MockHttpServletRequest 对象【请求对象】
② andDo：andDo(ResultHandler handler) throws Exception执行普通处理，例如 MockMvcResultHandlers的print() 方法用于 打印请求、响应及其他相关信息
③ andExpect：andExpect(ResultMatcher matcher) throws Exception
执行预期匹配。
例如：MockMvcResultMatchers.status().isOk() 预期响应成功

MockMvcResultMatchers.content().string(final String expectedContent) 指定预期的返回结果内容[字符串]

MockMvcResultMatchers.content().contentType(String contentType) 指定预期的返回结果的媒体类型

MockMvcResultMatchers.forwardedUrl(final String expectedUrl) 指定预期的请求的URL链接

MockMvcResultMathcers.redirectedUrl(final String expectedUrl) 指定预期的重定向的URL链接

注意：当有一项不满足时，则后续就不会进行。

④ andReturn

andReturn()
返回 MvcResult [请求访问结果]对象

⑤ getRequest

getRequest()
返回 MockHttpServletRequest [请求]对象

二、mybatis里的sql语句用法
（一）动态sql语句的增删改查操作
1.查询
<select id="findAll" parameterType="map" resultMap="studentMap">
        select id , name , sal
        from students
        <where>
            <if test="pid!=null" >
                and id = #{pid}    
            </if>
            <if test="pname!=null" >
                and name = #{pname}
            </if>
            <if test="psal!=null" >
                and sal = #{psal}
            </if>
        </where>
    </select>
</mapper>

2.插入
<sql id="key">
      <trim suffixOverrides=",">
        <if test="id!=null">
            id,
        </if>
        <if test="name!=null">
            name,
        </if>
        <if test="sal!=null">
            sal,
        </if>
      </trim>
    </sql>
    <!-- 定义第二个sql片段，第二个对应?，key属性值任意并且唯一 -->
    <sql id="value">
      <trim suffixOverrides=",">
        <if test="id!=null">
            #{id},
        </if>
        <if test="name!=null">
            #{name},
        </if>
        <if test="sal!=null">
            #{sal},
        </if>
      </trim>
    </sql>
    <!-- <include refid="key"/>和<include refid="value"/>表示引用上面sql片段 -->
    <insert id="insertStudent" parameterType="com.jpzhutech.entity.Student">
        insert into students(<include refid="key"/>) values(<include refid="value"/>);
    </insert>

3.删除
 <delete id="deleteStudent">
        delete from students where id in
        <!-- foreach用于迭代数组元素 
             open表示开始符号
             close表示结束符号
             seprator表示元素间的分割符
             items表示迭代的数组
        -->
        <foreach collection="array" open="(" close=")" separator="," item="ids">
            #{ids}
        </foreach>
    </delete>
 
 4.更新
 <update id="updateStudent" parameterType="map" >
        update students
        <set>
            <if test="pname!=null">
                name = #{pname},
            </if>
            <if test="psal!=null">
                sal = #{psal},
            </if>
        </set>
        where id = #{pid}
    </update>

（二）mybatis里的sql各类标签用法
一个普通的查询：
<select id="getStudentListLikeName" parameterType="StudentEntity" resultMap="studentResultMap">  
    SELECT * from STUDENT_TBL ST    
WHERE ST.STUDENT_NAME LIKE CONCAT(CONCAT('%', #{studentName}),'%')   
</select>  

1.if标签
如果studentName是null或空字符串，此语句很可能报错或查询结果为空。此时我们使用if动态sql语句先进行判断，如果值为null或等于空字符串，我们就不进行此条件的判断。
<select id=" getStudentListLikeName " parameterType="StudentEntity" resultMap="studentResultMap">  
    SELECT * from STUDENT_TBL ST   
    <if test="studentName!=null and studentName!='' ">  
        WHERE ST.STUDENT_NAME LIKE CONCAT(CONCAT('%', #{studentName}),'%')   
    </if>  
</select>  

2.where标签
参数studentName为null或’’，则或导致此sql组合成“WHERE AND”之类的关键字多余的错误SQL。
 这时我们可以使用where动态语句来解决。这个“where”标签会知道如果它包含的标签中有返回值的话，它就插入一个‘where’。此外，如果标签返回的内容是以AND 或OR 开头的，则它会剔除掉。
<select id="getStudentListWhere" parameterType="StudentEntity" resultMap="studentResultMap">  
    SELECT * from STUDENT_TBL ST   
    <where>  
        <if test="studentName!=null and studentName!='' ">  
            ST.STUDENT_NAME LIKE CONCAT(CONCAT('%', #{studentName}),'%')   
        </if>  
        <if test="studentSex!= null and studentSex!= '' ">  
            AND ST.STUDENT_SEX = #{studentSex}   
        </if>  
    </where>  
</select>  
 
 3.set标签
 当在update语句中使用if标签时，如果前面的if没有执行，则或导致逗号多余错误。使用set标签可以将动态的配置SET 关键字，和剔除追加到条件末尾的任何不相关的逗号。
 <update id="updateStudent" parameterType="StudentEntity">  
    UPDATE STUDENT_TBL   
    <set>  
        <if test="studentName!=null and studentName!='' ">  
            STUDENT_TBL.STUDENT_NAME = #{studentName},   
        </if>  
        <if test="studentSex!=null and studentSex!='' ">  
            STUDENT_TBL.STUDENT_SEX = #{studentSex},   
        </if>  
        <if test="studentBirthday!=null ">  
            STUDENT_TBL.STUDENT_BIRTHDAY = #{studentBirthday},   
        </if>  
        <if test="classEntity!=null and classEntity.classID!=null and classEntity.classID!='' ">  
            STUDENT_TBL.CLASS_ID = #{classEntity.classID}   
        </if>  
    </set>  
    WHERE STUDENT_TBL.STUDENT_ID = #{studentID};   
</update>  
 
 4，trim标签
  trim是更灵活的去处多余关键字的标签，他可以实践where和set的效果
  <select id="getStudentListWhere" parameterType="StudentEntity" resultMap="studentResultMap">  
    SELECT * from STUDENT_TBL ST   
    <trim prefix="WHERE" prefixOverrides="AND|OR">  
        <if test="studentName!=null and studentName!='' ">  
            ST.STUDENT_NAME LIKE CONCAT(CONCAT('%', #{studentName}),'%')   
        </if>  
        <if test="studentSex!= null and studentSex!= '' ">  
            AND ST.STUDENT_SEX = #{studentSex}   
        </if>  
    </trim>  
</select>  

5.choose标签
有时候我们并不想应用所有的条件，而只是想从多个选项中选择一个。MyBatis提供了choose 元素，按顺序判断when中的条件出否成立，如果有一个成立，则choose结束。当choose中所有when的条件都不满则时，则执行otherwise中的sql。类似于Java 的switch 语句，choose为switch，when为case，otherwise则为default。 if是与(and)的关系，而choose是或（or）的关系。
<select id="getStudentListChooseEntity" parameterType="StudentEntity" resultMap="studentResultMap">  
    SELECT * from STUDENT_TBL ST   
    <where>  
        <choose>  
            <when test="studentName!=null and studentName!='' ">  
                    ST.STUDENT_NAME LIKE CONCAT(CONCAT('%', #{studentName}),'%')   
            </when>  
            <when test="studentSex!= null and studentSex!= '' ">  
                    AND ST.STUDENT_SEX = #{studentSex}   
            </when>  
            <when test="studentBirthday!=null">  
                AND ST.STUDENT_BIRTHDAY = #{studentBirthday}   
            </when>  
            <when test="classEntity!=null and classEntity.classID !=null and classEntity.classID!='' ">  
                AND ST.CLASS_ID = #{classEntity.classID}   
            </when>  
            <otherwise>  
                   
            </otherwise>  
        </choose>  
    </where>  
</select>  

6.foreach标签
<select id="getStudentListByClassIDs" resultMap="studentResultMap">  
    SELECT * FROM STUDENT_TBL ST   
     WHERE ST.CLASS_ID IN    
     <foreach collection="list" item="classList"  open="(" separator="," close=")">  
        #{classList}   
     </foreach>      
</select>  
 
 
 
 


- Please forked this repo for practice
- Read Stories and finish follow requirements
- Design your API list and put it to root folder - `API.text`, including the JSON data format, HTTP METHOD, URI as well as status code. You can try https://editor.swagger.io for API design
- Design your E-R diagram and put it to root folder `E-R.jpg`
- Complement the stories by following 3 layers architecture (controller-service-repository) 
- Write API testing via MockMVC
- Write `Repository` via mybatis testing starter (There is an example for repository, feel free to replace them with your testing)
- Write other unit testing if needed
- Your commit message MUST follow the semantic message rule

### Stores

#### Story 1

As a manager, I want to create and list all parking boys. So that I can find someone to park cars for the customer.

AC1. I should be able to create parking boy to the system. The parking boy contains the following information:
employeeID: The employee id is a non-empty String representing the unique ID for a parking boy.

AC2. I should be able to list ALL the parking boys in the system. Each parking boy should include his employeeID.

#### Story 2

As a manager, I want to create and list all parking lots so that that parking boys can park cars into them.

AC1. I should be able to create a parking lot in the system. The parking lot contains the following information:
parkingLotID: The parking lot id is a non-empty String representing the unique ID for a parking lot.
capacity: The capacity of the parking lot. It is an integer from 1 - 100.

AC2. I should be able to list ALL parking lots in the system. Each parking lot should contain its parkingLotID, availablePositionCount and capacity of each parking lot.

#### Story 3

As a manager, I would like to associate parking lots to parking boys so that the parking boy can park cars in parking lots.

AC1. I should be able to associate one or more parking lots to a parking boy. Note that each parking lot can only be associated with one parking boy.

AC2. I should be able to display the parking boy and his/her managed parking lots. The parking boy should contain his/her employeeID and the associated parking lots should include their parkingLotID.
 
##  Practice Output & Submit

- submit your git repo url to field `answer`

## Hint

- create `Entity` to present your data structure
- create `Repository` for MyBatis integration 
- create `Mapper` under resources package 
- write sql statements 
- use `Repository` for your business to access to database

## How to write semantic commit message 

```text
feat: add hat wobble
^--^  ^------------^
|     |
|     +-> Summary in present tense.
|
+-------> Type: chore, docs, feat, fix, refactor, style, or test.
```

## How to use H2

- `schema.sql` will be loaded and init database when application is starting
- navigate to web console`http://localhost:8080/h2-console`
- put `jdbc:h2:mem:tws_persistence_db` in `JDBC URL` field
