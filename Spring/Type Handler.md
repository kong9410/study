# 타입 핸들러

MyBatis에서 Select된 Enum의 코드들을 Handler를 통해서 Enum으로 바꾸어준다.

```java
public interface CodeEnum {
    String getCode();
}
```
핸들러를 적용시키고자 하는 Enum의 인터페이스를 선언

```java
public enum MyType implements CodeEnum {
    CASH("01"),
    CARD("02"),
    POINT("03");
    
    private String code;
    
    MyType(String code) {
        this.code = code;
    }
    
    @Override
    private String getCode(
        return code;
    }
}
```
인터페이스를 상속받은 Enum 정의

```java
public class CodeEnumTypeHandler<E extends Enum<E>> implements TypeHandler<CodeEnum> {
    private Class<E> type;

    public CodeEnumTypeHandler(Class<E> type) {
        this.type = type;
    }

    @Override
    public void setParameter(PreparedStatement preparedStatement, int i, CodeEnum codeEnum, JdbcType jdbcType) throws
            SQLException {
        preparedStatement.setString(i, codeEnum.getCode());
    }

    @Override
    public CodeEnum getResult(ResultSet resultSet, String s) throws SQLException {
        return getCodeEnum(resultSet.getString(s));
    }

    @Override
    public CodeEnum getResult(ResultSet resultSet, int i) throws SQLException {
        return getCodeEnum(resultSet.getString(i));
    }

    @Override
    public CodeEnum getResult(CallableStatement callableStatement, int i) throws SQLException {
        return getCodeEnum(callableStatement.getString(i));
    }

    private CodeEnum getCodeEnum(String code) {
        try {
            CodeEnum[] enumConstants = (CodeEnum[])type.getEnumConstants();
            return Arrays.stream(enumConstants)
                    .filter(codeEnum -> codeEnum.getCode().equals(code))
                    .findFirst()
                    .orElse(null);
        } catch (Exception e) {
            throw new TypeException(new StringBuilder("Can't make enum object '")
                            .append(type)
                            .append("'\n")
                            .append(e)
                            .toString());
        }
    }
}
```
TypeHandler 인터페이스를 상속받은 핸들러를 선언
쿼리문을 통해 CodeEnum의 code와 일치하는 enum을 반환하게함

```xml
<resultMap id="resultEnum" type="MyDto">
    <result column="type" property="type" typeHandler="com.navercorp.type.handler.CodeEnumTypeHandler"/>
</resultMap>
<select id="selectEnumType" resultMap="resultEnum">
    SELECT type
    FROM test
</select>
```