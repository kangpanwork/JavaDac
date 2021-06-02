项目中`CRUD`，写好`XML`很重要
个人经常使用的模板
- 定义全属性的 `ResultMap`
- 使用 `<sql></sql>` 抽离片段 
`<include></include>` 重复使用 
    - 表名 
    ```
    <sql id="tableName">
        表名
    </sql>
    ```
    - order by 条件
    ```
     <sql id="condition">
        表字段
    </sql>
    ```
    - 表字段
    ```
    <sql id="allFields">
        id,
        name,
        ...
        表字段
    </sql>
    ```
    - 表字段赋值
    ```
    <sql id="allValues">
        #{id,jdbcType = BIGINT},...
    </sql>
    ```
    - 主键赋值
    ```
    <sql id="uniquekeyField">
        #{id,jdbcType = BIGINT}；
    </sql>
    ```
  
    - 条件赋值
    ```
   <sql id="setValues">
     <if test="id != null">
           id = #{id,jdbcType = BIGINT}
      </if>
      ...
    </sql>
    ```
    - 搜索条件赋值 `prefixOverrides` 当第一个条件不满足，自动去除`AND 或者 OR`  
    ```
    <sql id = "searchFields">
        trim prefix="where" prefixOverrides="and|or">  
            <if test="name!=null">  
                AND name like concat('%',#{name,jdbcType=VARCHAR},'%')
            </if>  
        </trim>  
    </sql>
    ```
    - 分页查询
    ```
        <select id = "findByPage" resultMap = "resultMap">
            select <include refid = "allFields" />
            from 表名
            <include refid = "searchFields">
            limit #{开始},#{分页大小}
        </select>
    ```
    - 查询数量
     ```
   <select id = "findByCount">
       select count(1) from 表名
       <include refid = "searchFields">
    </select>
     ```
    - 查询所有字段
    ```
    <select id="findAllFields">
       select <include refid = "allFields">
       from 表名
       <include refid = "allValues">
    </select>
    ```
    - 根据 `id` 查询，复用 `uniquekeyField`
       
    ```
    <select id="findById">
    </select>
    ```
    - 批量查询
    ```
<select id="batchSelect" resultType="" parameterType="java.util.List">
        select <include refid = "allFields">
        from 表名 
        <where>
           id in
            <trim prefix="(" suffix=")">
                <foreach collection="list" item="item"  index="index"  separator=",">
                    <include refid = "setValues">
                </foreach>
            </trim>
        </where>
        order by <include refid = "condition">
    </select>
    ```
    ```
 <select id="findByArray" parameterType="Object[]" resultType="">
      select <include refid = "allFields">
      from 表名
        <where>
          <if test="array!=null">
              <foreach collection="array" index="index" item="item"
                           open="and id in("separator=","close=")">
                      #{item.id}
              </foreach>
        </if>
        </where>
</select>
    ```
    - 插入
        ```
   <insert id = "insert">
       insert into 表名
       <include refid = "allFields">
       values
       <include refid = "setValues">
    </insert>
     ```
    - 批量插入
    ```
   <insert id="batchInsert" parameterType="java.util.List" useGeneratedKeys="true" keyProperty="id">
        insert into 表名 
        <include refid = "allFields">
        values
        <!--mybatis 参数映射为list @Param 可以指定入参名称-->
        <foreach collection="list" item="item" index="index" separator=",">
        <include refid = "setValues">
        </foreach>
    </insert>
     ```
    - 批量主键删除
    ```
     <delete id="batchDelete" parameterType="long">
        delete from 表名 where id in
        <foreach collection="array" item="item" open="(" separator="," close=")">
        <include refid = "uniquekeyField">
        </foreach>
    </delete>
    ```
    - 主键删除
    ```
     <delete id="delete" parameterType="long">
        delete from 表名 where id = <include refid = "uniquekeyField">
    </delete>
    ```
    - 批量删除 `map.put("ids","1,2,3");map.put("name","kangpan")` 
    ```
    <delete id="batchDelete" parameterType="java.util.Map">
      delete from 表名
        <where>
            <if test="ids != null">
                id in (#{ids,jdbcType=BIGINT})
            </if>
            <if test="name != null && name != ''">
                and name = #{name,jdbcType=VARCHAR}
            </if>
        </where>
    </delete>
    ```
    - 更新
      ```
     <update id="update" parameterType="">
        update 表名 
        <set>
          <include refid = "setValues">
        </set>
        where t.id = <include refid = "uniquekeyField">
    </update>
      ```
      ```
  <update id="update" parameterType="">  
        update 表名  
        <trim prefix="SET" suffixOverrides=",">  
        <include refid = "setValues">
        </trim>  
        where id = <include refid = "uniquekeyField"> 
    </update>
      ```
    - 批量更新
     ```
     <update id="batchUpdate" parameterType="java.util.List">
        <foreach collection="list" item="item" index="index" open="" close="" separator=";">
            update 表名 
            <set>
                <include refid = "setValues">
            </set>
            where t.id = <include refid = "uniquekeyField">
        </foreach>
    </update>
     ```
     [了解更多](https://mybatis.org/mybatis-3/zh/)