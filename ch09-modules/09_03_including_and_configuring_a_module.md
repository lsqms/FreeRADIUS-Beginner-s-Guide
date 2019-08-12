# 包括和配置模块
某些模块只能用于特定功能。 pap模块就是这样，仅用于身份验证。 相反，sql模块可用于授权，会话检查以及记帐。 这完全取决于模块作者所包含的功能。
sql模块（rlm_sql.so）使用子模块。 这创建了一个抽象层。 根据主sql模块的配置方式，它将使用特定的子模块与某种类型的数据库进行交互。 子模块可用于连接MySQL（rml_sql_mysql.so），PostgreSQL（rlm_sql_postgresql.so），Microsoft SQL Server（rlm_sql_iodbc.so）和Oracle（rlm_sql_oracle.so）数据库。
反过来，sql子模块也可以配置为微调其行为。