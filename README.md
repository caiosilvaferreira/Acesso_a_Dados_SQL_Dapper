# Acesso à dados com .NET, C#, Dapper e SQL Server

OBS: CASO TENHA DUVIDA DE QUALQUER UM DOS CODIGOS CONSULTAR NO CHATGPT

 O [ADO.NET](http://ADO.NET) É O CORE DE ACESSO AOS DADOS DA MICROSOFT.

RESUMINDO ESSE TIPO DE CONEXÃO É USADO NO ADO.NET, QUE É MAIS COMPLICADA, OS DEVS UTILIZAR MAIS A CONEXÃO COM DAPPER, MENOS CODIGO E A PERFOMACE QUASE A MESMA.

---

# CÓDIGO

```sql
using System;
using Microsoft.Data.SqlClient;

namespace BaltaDataAccess {
    class Program {
        static void Main(string[] args) {
            const string connectionString = "Server=localhost,1433;Database=balta;User ID=sa;Password=1q2w3e4r@#$; TrustServerCertificate=True";

            using (var connection = new SqlConnection(connectionString))
                Console.WriteLine("Conectado");
                connection.Open();

                using (var command = new SqlCommand()) {
                    command.Connection = connection;
                    command.CommandType = System.Data.CommandType.Text;
                    command.CommandText = "SELECT [Id], [Title] FROM [Category]";

                    var reader = command.ExecuteReader();
                    while (reader.Read()) {
                        Console.WriteLine($"{reader.GetGuid(0)} - {reader.GetString(1)}");
                    }
                }
            }

            Console.WriteLine("Hello World!");
        }
    }
}

```

---

# USEI O PROPRIO TERMINAL DO VISUAL STUDIO PARA INSTALAR O DAPPER COM SEGUINTE CODIGO

```sql
dotnet add package dapper

```

DEPOIS DISSO ELE FEZ A INSTALAÇÃO, E CONFERIR NO CSPROJ PARA VER SE REALMENTE INSTALOU

---

# ESSE É MEU PRIMEIRO CODIGO USANDO O DAPPER PARA PUXAR OS DADOS NO BANCO

```sql
using System;
using conectionSql.NewFolder;
using Dapper;
using Microsoft.Data.SqlClient;

namespace BaltaDataAccess {
    class Program {
        static void Main(string[] args) {
            const string connectionString = "Server=localhost,1433;Database=balta;User ID=sa;Password=1q2w3e4r@#$; TrustServerCertificate=True";

            using (var connection = new SqlConnection(connectionString)) {

                var categories = connection.Query<Category>("SELECT [Id], [Title] FROM [Category]");

                foreach (var category in categories) {

                    Console.WriteLine($"{category.Id} - {category.Title}");
                }
            }
        }
    }
}

```

---

# tivemos que criar uma classe com as propriedade do banco de dados

```sql
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace conectionSql.Models {
    public class Category {

        public Guid Id { get; set; }
        public String Title { get; set; }

    }
}
```

RESULTADO: 

![Untitled](Acesso%20a%CC%80%20dados%20com%20NET,%20C#,%20Dapper%20e%20SQL%20Server%207317555570484cceb98e71385128ae94/Untitled.png)

---

# UMA COISA QUE CONSEGUIMOS FAZER É RENOMEAR O NOME DE CADA PARAMETRO, CASO A GENTE QUEIRA

![Untitled](Acesso%20a%CC%80%20dados%20com%20NET,%20C#,%20Dapper%20e%20SQL%20Server%207317555570484cceb98e71385128ae94/Untitled%201.png)

E NA CLASSE TBM 

![Untitled](Acesso%20a%CC%80%20dados%20com%20NET,%20C#,%20Dapper%20e%20SQL%20Server%207317555570484cceb98e71385128ae94/Untitled%202.png)

SE DEIXAMOS COM NOME DIFERENTE O DAPPER VAI GERAR ERRO, MAS ISSO É SO UMA DICA

---

# NESSE CODIGO FIZEMOS UM INSERT INTO PARA MODIFICAR ALGUNS PARAMETROS EMBAIXO TEM O RESULTADO DO CODIGO

```sql
using System;
using conectionSql.Models;
using Dapper;
using Microsoft.Data.SqlClient;

namespace BaltaDataAccess {
    class Program {
        static void Main(string[] args) {
            const string connectionString = "Server=localhost,1433;Database=balta;User ID=sa;Password=1q2w3e4r@#$; TrustServerCertificate=True";

            var category = new Category();
            category.Id= Guid.NewGuid();
            category.Title = "Amazon AWS";
            category.Url = "amazon";
            category.Description ="Categoria destinada a serviços do AWS";
            category.Order = 8;
            category.Summary = "AWS Cloud";
            category.Featured = false;
            var insertSql = @"INSERT INTO
                    [Category]
                VALUES
                    (
                    
                    @Id, 
                    @Title,     
                    @Url, 
                    @Summary, 
                    @Order, 
                    @Description,
                    @Featured)";  /* nunca concatenar strings no select com o C#, pois facilita que o usuario possa fazer um insert pela url ou receber ataque hackers*/

            using (var connection = new SqlConnection(connectionString)) {
                //connection.Execute retornar um int de quantidade de linhas modificadas
                var rows = connection.Execute(insertSql, new { 
                    category.Id,
                    category.Title,
                    category.Url,
                    category.Summary,
                    category.Order,
                    category.Description,
                    category.Featured

                });
                Console.WriteLine($"{rows} linhas inseridas");
                // mostramos quantas linhas são modificadas
                var categories = connection.Query<Category>("SELECT [Id], [Title] FROM [Category]");

                foreach (var item in categories) {

                    Console.WriteLine($"{item.Id} - {item.Title}");
                }
            }
        }
    }
}
```

RESULTADO

![Untitled](Acesso%20a%CC%80%20dados%20com%20NET,%20C#,%20Dapper%20e%20SQL%20Server%207317555570484cceb98e71385128ae94/Untitled%203.png)

---

# PARA QUE TUDO ISSO POSSA ACONTECER O DOCKER TEM QUE ESTA FUNCIONANDO E RODANDO O SERVIDOR DE CONEXÃO

![Untitled](Acesso%20a%CC%80%20dados%20com%20NET,%20C#,%20Dapper%20e%20SQL%20Server%207317555570484cceb98e71385128ae94/Untitled%204.png)

---

# AGORA COM CODIGO MAIS ORGANIZADO

```sql
using System;
using conectionSql.Models;
using Dapper;
using Microsoft.Data.SqlClient;

namespace BaltaDataAccess {
    class Program {
        static void Main(string[] args) {
            const string connectionString = "Server=localhost,1433;Database=balta;User ID=sa;Password=1q2w3e4r@#$; TrustServerCertificate=True";

            using (var connection = new SqlConnection(connectionString)) {

                UpdateCategory(connection);    // aqui ele chama os dois metodos criados abaixo e o codigo ficou bem menor
                ListCategories(connection);
               // CreateCategory(connection);
            }
        }
        static void ListCategories(SqlConnection connection) {

            var categories = connection.Query<Category>("SELECT [Id], [Title] FROM [Category]");

            foreach (var item in categories) {

                Console.WriteLine($"{item.Id} - {item.Title}");
            }

        }

        static void CreateCategory(SqlConnection connection) {

            var category = new Category();
            category.Id= Guid.NewGuid();
            category.Title = "Amazon AWS";
            category.Url = "amazon";
            category.Description ="Categoria destinada a serviços do AWS";
            category.Order = 8;
            category.Summary = "AWS Cloud";
            category.Featured = false;
            var insertSql = @"INSERT INTO
                    [Category]
                VALUES
                    (

                    @Id,
                    @Title,
                    @Url,
                    @Summary,
                    @Order,
                    @Description,
                    @Featured)";

            var rows = connection.Execute(insertSql, new {
                category.Id,
                category.Title,
                category.Url,
                category.Summary,
                category.Order,
                category.Description,
                category.Featured

            });
            Console.WriteLine($"{rows} linhas inseridas");

        }

        static void UpdateCategory(SqlConnection connection) {

            var updateQuery = "UPDATE [Category] SET [Title]=@title WHERE [Id]=@id ";
            var rows = connection.Execute(updateQuery, new {
                id= new Guid("32b5b133-f75e-4c5b-8915-e6b661e25bec"), // como o id é diferente nao teve alteração de nada, caso seja o mesmo id ele faz o update
                title = "Frontend 2021"
            });

            Console.WriteLine($"{rows} registros atualizados");
        }
    }

}

```

---

# CODIGO COM CREATE UPDATE E DELETE

```sql
using conectionSql.Models;
using Dapper;
using Microsoft.Data.SqlClient;

namespace BaltaDataAccess {
    class Program {
        static void Main(string[] args) {
            const string connectionString = "Server=localhost,1433;Database=balta;User ID=sa;Password=1q2w3e4r@#$; TrustServerCertificate=True";

            using (var connection = new SqlConnection(connectionString)) {
                // CreateCategory(connection);

                 //UpdateCategory(connection);  DEIXAMOS COMENTADO PARA EXECUTAR UM DE CADA VEZ
                 DeleteCategory(connection);
                //ListCategories(connection);
                //GetCategory(connection);

            }
        }

        static void ListCategories(SqlConnection connection) {
            var categories = connection.Query<Category>("SELECT [Id], [Title] FROM [Category]");
            foreach (var item in categories) {
                Console.WriteLine($"{item.Id} - {item.Title}");
            }
        }

        static void GetCategory(SqlConnection connection) {
            var category = connection
                .QueryFirstOrDefault<Category>(
                    "SELECT TOP 1 [Id], [Title] FROM [Category] WHERE [Id]=@id",
                    new {
                        id = "af3407aa-11ae-4621-a2ef-2028b85507c4"
                    });
            Console.WriteLine($"{category.Id} - {category.Title}");

        }

        static void CreateCategory(SqlConnection connection) {
            var category = new Category();
            category.Id = Guid.NewGuid();
            category.Title = "Amazon AWS";
            category.Url = "amazon";
            category.Description = "Categoria destinada a serviços do AWS";
            category.Order = 8;
            category.Summary = "AWS Cloud";
            category.Featured = false;

            var insertSql = @"INSERT INTO
                    [Category]
                VALUES(
                    @Id,
                    @Title,
                    @Url,
                    @Summary,
                    @Order,
                    @Description,
                    @Featured)";

            var rows = connection.Execute(insertSql, new {
                category.Id,
                category.Title,
                category.Url,
                category.Summary,
                category.Order,
                category.Description,
                category.Featured
            });
            Console.WriteLine($"{rows} linhas inseridas");
        }

        static void UpdateCategory(SqlConnection connection) {
            var updateQuery = "UPDATE [Category] SET [Title]=@title WHERE [Id]=@id";
            var rows = connection.Execute(updateQuery, new {
                id = new Guid("af3407aa-11ae-4621-a2ef-2028b85507c4"),
                title = "Frontend 2021"
            });

            Console.WriteLine($"{rows} registros atualizadas");
        }

        static void DeleteCategory(SqlConnection connection) {
            var deleteQuery = "DELETE [Category] WHERE [Id]=@id";
            var rows = connection.Execute(deleteQuery, new {
                id = new Guid("ffb43c2f-e440-4259-aa0c-14fd4932f234"),
            });

            Console.WriteLine($"{rows} registros excluídos");
        }

    }
}

```

---

# EXECUTE MANY, TEM A FUNÇÃO DE ALTERAR VARIAS LINHAS SIMULTANEOS, PODE SER USADO PARA CREATE, UPDATE OU DELETE.

```sql
using conectionSql.Models;
using Dapper;
using Microsoft.Data.SqlClient;

namespace BaltaDataAccess {
    class Program {
        static void Main(string[] args) {
            const string connectionString = "Server=localhost,1433;Database=balta;User ID=sa;Password=1q2w3e4r@#$; TrustServerCertificate=True";

            using (var connection = new SqlConnection(connectionString)) {

                 //CreateManyCategory(connection);

                //ListCategories(connection);
                //GetCategory(connection);

                ExecuteProcedure(connection);

            }
        }

        static void ListCategories(SqlConnection connection) {
            var categories = connection.Query<Category>("SELECT [Id], [Title] FROM [Category]");
            foreach (var item in categories) {
                Console.WriteLine($"{item.Id} - {item.Title}");
            }
        }

        static void CreateManyCategory(SqlConnection connection) {
            var category = new Category();
            category.Id = Guid.NewGuid();
            category.Title = "Amazon AWS";
            category.Url = "amazon";
            category.Description = "Categoria destinada a serviços do AWS";
            category.Order = 8;
            category.Summary = "AWS Cloud";
            category.Featured = false;

            var category2 = new Category();
            category2.Id = Guid.NewGuid();
            category2.Title = "Categoria nova";
            category2.Url = "categoria-nova";
            category2.Description = "Categoria nova";
            category2.Order = 9;
            category2.Summary = "Categoria";
            category2.Featured = true;

            var insertSql = @"INSERT INTO
                    [Category]
                VALUES(
                    @Id,
                    @Title,
                    @Url,
                    @Summary,
                    @Order,
                    @Description,
                    @Featured)";

            var rows = connection.Execute(insertSql, new[]{
                new{
                category.Id,
                category.Title,
                category.Url,
                category.Summary,
                category.Order,
                category.Description,
                category.Featured
                },
                new
                {
                category2.Id,
                category2.Title,
                category2.Url,
                category2.Summary,
                category2.Order,
                category2.Description,
                category2.Featured
            }
            });
            Console.WriteLine($"{rows} linhas inseridas");
        }

    }
}

```

---

# ESSE CODIGO É RESPONSAVEL EM EXECUTAR UM PROCEDURE, UMA PROCEDURE É UMA IMAGEM DE UMA QUERY ARMAZENADA, NO CASO O QUE FOI CRIADO É UM SCRIPT DELETANDO OS DADOS DO BANCO STUDENT

```sql
using conectionSql.Models;
using Dapper;
using Microsoft.Data.SqlClient;

namespace BaltaDataAccess {
    class Program {
        static void Main(string[] args) {
            const string connectionString = "Server=localhost,1433;Database=balta;User ID=sa;Password=1q2w3e4r@#$; TrustServerCertificate=True";

            using (var connection = new SqlConnection(connectionString)) {
                // CreateCategory(connection);
                 //CreateManyCategory(connection);
                 //UpdateCategory(connection);
                // DeleteCategory(connection);
                //ListCategories(connection);
                //GetCategory(connection);

                ExecuteProcedure(connection);

            }
        }

        static void ExecuteProcedure(SqlConnection connection) {
            var procedure = "[spDeleteStudent]";
            var pars = new { StudentId = "17097f1a-56b3-4a0d-a42c-c7a548c96b6f" };
            var affectedRows = connection.Execute(procedure, pars, commandType: System.Data.CommandType.StoredProcedure);

            Console.WriteLine($"{affectedRows} linhas afetadas");
        }
    }
}

```

---

# EXECUTANDO UMA PROCEDURE

```sql
using conectionSql.Models;
using Dapper;
using Microsoft.Data.SqlClient;
using System.Data;

namespace BaltaDataAccess {
    class Program {
        static void Main(string[] args) {
            const string connectionString = "Server=localhost,1433;Database=balta;User ID=sa;Password=1q2w3e4r@#$; TrustServerCertificate=True";

            using (var connection = new SqlConnection(connectionString)) {
                // CreateCategory(connection);
                //CreateManyCategory(connection);
                //UpdateCategory(connection);
                // DeleteCategory(connection);
                //ListCategories(connection);
                //GetCategory(connection);
                //ExecuteProcedure(connection);
                ExecuteReadProcedure(connection);
            }
        }

        static void ExecuteReadProcedure(SqlConnection connection) {
            var procedure = "[spGetCoursesByCategory]";
            var pars = new { CategoryId = "09ce0b7b-cfca-497b-92c0-3290ad9d5142" };
            var courses = connection.Query(
                procedure,
                pars,
                commandType: CommandType.StoredProcedure);

            foreach (var item in courses) {
                Console.WriteLine(item.Title);
            }
        }
    }
}

```

---

# EXECUTE SCALAR TEM A VANTAGEM DE MOSTRAR O ID DA TABELA QUE FOI ALTERADA, ENQUANTO AS OUTRAS FUNÇÕES RETORNA SOMENTE UM INT, POR ISSO NOS CODIGO ANTIGO ESTAVAMOS USANDO FOREACH

```sql
using conectionSql.Models;
using Dapper;
using Microsoft.Data.SqlClient;
using System.Data;

namespace BaltaDataAccess {
    class Program {
        static void Main(string[] args) {
            const string connectionString = "Server=localhost,1433;Database=balta;User ID=sa;Password=1q2w3e4r@#$; TrustServerCertificate=True";

            using (var connection = new SqlConnection(connectionString)) {
                // CreateCategory(connection);
                //CreateManyCategory(connection);
                //UpdateCategory(connection);
                // DeleteCategory(connection);
                //ListCategories(connection);
                //GetCategory(connection);
                //ExecuteProcedure(connection);
                //ExecuteReadProcedure(connection);
                ExecuteScalar(connection);
            }
        }

        static void ExecuteScalar(SqlConnection connection) {

            var category = new Category();
            category.Title = "Amazon AWS";
            category.Url = "amazon";
            category.Description = "Categoria destinada a serviços do AWS";
            category.Order = 8;
            category.Summary = "AWS Cloud";
            category.Featured = false;

            var insertSql = @"INSERT INTO
                    [Category]
                OUTPUT inserted.[Id]
                VALUES(
                    NEWID(),
                    @Title,
                    @Url,
                    @Summary,
                    @Order,
                    @Description,
                    @Featured)
                    "; // OUTPUT vai localizar o id do select modificado

            var id = connection.ExecuteScalar<Guid>(insertSql, new {
                category.Title,
                category.Url,
                category.Summary,
                category.Order,
                category.Description,
                category.Featured
            });
            Console.WriteLine($" A categoria inserida foi :{id}");
        }
    }
}

```

---

# CODIGO DE UMA VIEW CRIADA NO DATA STUDIO

```sql
using conectionSql.Models;
using Dapper;
using Microsoft.Data.SqlClient;
using System.Data;

namespace BaltaDataAccess {
    class Program {
        static void Main(string[] args) {
            const string connectionString = "Server=localhost,1433;Database=balta;User ID=sa;Password=1q2w3e4r@#$; TrustServerCertificate=True";

            using (var connection = new SqlConnection(connectionString)) {
                // CreateCategory(connection);
                //CreateManyCategory(connection);
                //UpdateCategory(connection);
                // DeleteCategory(connection);
                //ListCategories(connection);
                //GetCategory(connection);
                //ExecuteProcedure(connection);
                //ExecuteReadProcedure(connection);
                //ExecuteScalar(connection);
                ReadView(connection);
            }
        }

        static void ReadView(SqlConnection connection) {
            var sql = "SELECT * FROM [vwCourses]";
            var courses = connection.Query(sql);
            foreach (var item in courses) {
                Console.WriteLine($"{item.Id} - {item.Title}");
            }

        }

    }
}

```

---

# USANDO O ONE TO ONE

ELE FAZ BASICAMENTE UM JOIN DE UM PARA UM, PEGA UMA INFORMAÇÃO DE UMA COLUNA E FAZ O JOIN COM UMA COLUNA DA OUTRA TABELA

```sql
using conectionSql.Models;
using Dapper;
using Microsoft.Data.SqlClient;
using System.Data;

namespace BaltaDataAccess {
    class Program {
        static void Main(string[] args) {
            const string connectionString = "Server=localhost,1433;Database=balta;User ID=sa;Password=1q2w3e4r@#$; TrustServerCertificate=True";

            using (var connection = new SqlConnection(connectionString)) {
                // CreateCategory(connection);
                //CreateManyCategory(connection);
                //UpdateCategory(connection);
                // DeleteCategory(connection);
                //ListCategories(connection);
                //GetCategory(connection);
                //ExecuteProcedure(connection);
                //ExecuteReadProcedure(connection);
                //ExecuteScalar(connection);
                //ReadView(connection);
                OneToOne(connection);
            }
        }

        static void OneToOne(SqlConnection connection) {

            var sql = @"
                SELECT * FROM [CareerItem] INNER JOIN [Course]
                ON [CareerItem].[CourseId] = [Course].[Id] ";

            var items = connection.Query<CareerItem, Course, CareerItem>(sql, (careerItem, course) => {

                careerItem.Course = course;
                return careerItem;
            }, splitOn:"Id");

            foreach(var item in items) {
                Console.WriteLine($"{item.Title} - Curso: {item.Course.Title}");
            }
        }

    }
}

```

TIVE QUE CRIAR MAIS DUAS CLASSES, PARA PUXAR AS INFORMAÇÕES

```sql
namespace conectionSql.Models {
    public class Course {

        public Guid Id { get; set; }

        public string Title { get; set; }
    }
}

```

```sql
namespace conectionSql.Models {
    public class CareerItem {

        public Guid Id { get; set; }

        public string Title { get; set; }

        public Course Course { get; set; }
    }
}

```

![Acesso%20a%CC%80%20dados%20com%20NET,%20C#,%20Dapper%20e%20SQL%20Server%207317555570484cceb98e71385128ae94/Untitled%205.png](Acesso%20a%CC%80%20dados%20com%20NET,%20C#,%20Dapper%20e%20SQL%20Server%207317555570484cceb98e71385128ae94/Untitled%205.png)

---

USANDO O ONE TO MANY

TRADUZINDO UM PARA TODOS, ELE CONSEGUE SELECIONA UMA COLUNA NO BANCO E PEGA TODAS INFORMAÇÕES DE UM OUTRO BANCO

```sql
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using conectionSql.Models;
using Dapper;
using Microsoft.Data.SqlClient;

namespace BaltaDataAccess {
    class Program {
        static void Main(string[] args) {
            const string connectionString = "Server=localhost,1433;Database=balta;User ID=sa;Password=1q2w3e4r@#$;TrustServerCertificate=true";

            using (var connection = new SqlConnection(connectionString)) {
                // CreateCategory(connection);
                // CreateManyCategory(connection);
                // UpdateCategory(connection);
                // DeleteCategory(connection);
                // ListCategories(connection);
                // GetCategory(connection);
                // ExecuteProcedure(connection);
                // ExecuteReadProcedure(connection);
                // ExecuteScalar(connection);
                // ReadView(connection);
                // OneToOne(connection);
                 OneToMany(connection);
                // QueryMutiple(connection);
                // SelectIn(connection);
                // Like(connection, "backend");
                // Transaction(connection);
            }
        }

        static void OneToMany(SqlConnection connection) {
            var sql = @"
                SELECT
                    [Career].[Id],
                    [Career].[Title],
                    [CareerItem].[CareerId],
                    [CareerItem].[Title]
                FROM
                    [Career]
                INNER JOIN
                    [CareerItem] ON [CareerItem].[CareerId] = [Career].[Id]
                ORDER BY
                    [Career].[Title]";

            var careers = new List<Career>();
            var items = connection.Query<Career, CareerItem, Career>(
                sql,
                (career, item) => {
                    var car = careers.Where(x => x.Id == career.Id).FirstOrDefault();
                    if (car == null) {
                        car = career;
                        car.Items.Add(item);
                        careers.Add(car);
                    } else {
                        car.Items.Add(item);
                    }

                    return career;
                }, splitOn: "CareerId");

            foreach (var career in careers) {
                Console.WriteLine($"{career.Title}");
                foreach (var item in career.Items) {
                    Console.WriteLine($" - {item.Title}");
                }
            }
        }

    }
}

```

---

# USANDO QUERY MULTIPLE

A FUNÇÃO DELE É PARA USAR VARIOS SELECT SIMULTANEOS NO CODIGO E PEGA AS INFORMAÇÕES DESEJADAS

```sql
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using conectionSql.Models;
using Dapper;
using Microsoft.Data.SqlClient;

namespace BaltaDataAccess {
    class Program {
        static void Main(string[] args) {
            const string connectionString = "Server=localhost,1433;Database=balta;User ID=sa;Password=1q2w3e4r@#$;TrustServerCertificate=true";

            using (var connection = new SqlConnection(connectionString)) {
                // CreateCategory(connection);
                // CreateManyCategory(connection);
                // UpdateCategory(connection);
                // DeleteCategory(connection);
                // ListCategories(connection);
                // GetCategory(connection);
                // ExecuteProcedure(connection);
                // ExecuteReadProcedure(connection);
                // ExecuteScalar(connection);
                // ReadView(connection);
                // OneToOne(connection);
                // OneToMany(connection);
                 QueryMutiple(connection);
                // SelectIn(connection);
                // Like(connection, "backend");
                // Transaction(connection);
            }
        }

        static void QueryMutiple(SqlConnection connection) {

            var query = "SELECT * FROM [Category]; SELECT * FROM [Course] ";

            using (var multi = connection.QueryMultiple(query)) {

                var categories = multi.Read<Category>();
                var courses = multi.Read<Course>();

                foreach(var item in categories) {

                    Console.WriteLine(item.Title);
                }

                foreach (var item in courses) {

                    Console.WriteLine(item.Title);
                }
            }
        }

    }
}

```

---

# USANDO SELECT IN

```csharp
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using conectionSql.Models;
using Dapper;
using Microsoft.Data.SqlClient;

namespace BaltaDataAccess {
    class Program {
        static void Main(string[] args) {
            const string connectionString = "Server=localhost,1433;Database=balta;User ID=sa;Password=1q2w3e4r@#$;TrustServerCertificate=true";

            using (var connection = new SqlConnection(connectionString)) {
                // CreateCategory(connection);
                // CreateManyCategory(connection);
                // UpdateCategory(connection);
                // DeleteCategory(connection);
                // ListCategories(connection);
                // GetCategory(connection);
                // ExecuteProcedure(connection);
                // ExecuteReadProcedure(connection);
                // ExecuteScalar(connection);
                // ReadView(connection);
                // OneToOne(connection);
                // OneToMany(connection);
                // QueryMutiple(connection);
                // SelectIn(connection);
                // Like(connection, "backend");
                // Transaction(connection);
                selectIn(connection);

            }
        }

        static void selectIn(SqlConnection connection) {

            var query = @"SELECT  * FROM career where [id] in @Id";

            var items = connection.Query<Career>(query, new {
                Id = new[] {
                    "e6730d1c-6870-4df3-ae68-438624e04c72",
                    "92d7e864-bea5-4812-80cc-c2f4e94db1af"
                }});
            foreach (var item in items) {

                Console.WriteLine(item.Title);
            }
        }

    }
}

```

---

# USANDO LIKE

```csharp
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using conectionSql.Models;
using Dapper;
using Microsoft.Data.SqlClient;

namespace BaltaDataAccess {
    class Program {
        static void Main(string[] args) {
            const string connectionString = "Server=localhost,1433;Database=balta;User ID=sa;Password=1q2w3e4r@#$;TrustServerCertificate=true";

            using (var connection = new SqlConnection(connectionString)) {
                // CreateCategory(connection);
                // CreateManyCategory(connection);
                // UpdateCategory(connection);
                // DeleteCategory(connection);
                // ListCategories(connection);
                // GetCategory(connection);
                // ExecuteProcedure(connection);
                // ExecuteReadProcedure(connection);
                // ExecuteScalar(connection);
                // ReadView(connection);
                // OneToOne(connection);
                // OneToMany(connection);
                // QueryMutiple(connection);
                // SelectIn(connection);
                // Like(connection, "backend");
                // Transaction(connection);
                //selectIn(connection);
                Like(connection);
            }
        }

        static void Like(SqlConnection connection) {

            var term = "api";
            var query = @"SELECT  * FROM [Course] where [Title] LIKE @exp";

            var items = connection.Query<Course>(query, new
            {
                exp =$"%{term}%"
            });
            foreach (var item in items) {

                Console.WriteLine(item.Title);
            }
        }
    }
}

```

---

# USANDO TRANSACTION

```sql
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;

using conectionSql.Models;
using Dapper;

using Microsoft.Data.SqlClient;

namespace BaltaDataAccess {
    class Program {
        static void Main(string[] args) {
            const string connectionString = "Server=localhost,1433;Database=balta;User ID=sa;Password=1q2w3e4r@#$; TrustServerCertificate=true";

            using (var connection = new SqlConnection(connectionString)) {
                // CreateCategory(connection);
                // CreateManyCategory(connection);
                // UpdateCategory(connection);
                // DeleteCategory(connection);
                // ListCategories(connection);
                // GetCategory(connection);
                // ExecuteProcedure(connection);
                // ExecuteReadProcedure(connection);
                // ExecuteScalar(connection);
                // ReadView(connection);
                // OneToOne(connection);
                // OneToMany(connection);
                // QueryMutiple(connection);
                // SelectIn(connection);
                // Like(connection, "backend");
                 Transaction(connection);
            }
        }

        static void Transaction(SqlConnection connection) {
            var category = new Category();
            category.Id = Guid.NewGuid();
            category.Title = "Minha categoria que não";
            category.Url = "amazon";
            category.Description = "Categoria destinada a serviços do AWS";
            category.Order = 8;
            category.Summary = "AWS Cloud";
            category.Featured = false;

            var insertSql = @"INSERT INTO
                    [Category]
                VALUES(
                    @Id,
                    @Title,
                    @Url,
                    @Summary,
                    @Order,
                    @Description,
                    @Featured)";

            connection.Open();
            using (var transaction = connection.BeginTransaction()) {
                var rows = connection.Execute(insertSql, new {
                    category.Id,
                    category.Title,
                    category.Url,
                    category.Summary,
                    category.Order,
                    category.Description,
                    category.Featured
                }, transaction);

                transaction.Commit();
                // transaction.Rollback();

                Console.WriteLine($"{rows} linhas inseridas");
            }
        }
    }
}

```

---

# CODIGO COMPLETO DO MODULO 3

```csharp
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;

using conectionSql.Models;
using Dapper;

using Microsoft.Data.SqlClient;

namespace BaltaDataAccess {
    class Program {
        static void Main(string[] args) {
            const string connectionString = "Server=localhost,1433;Database=balta;User ID=sa;Password=1q2w3e4r@#$; TrustServerCertificate=true";

            using (var connection = new SqlConnection(connectionString)) {
                // CreateCategory(connection);
                // CreateManyCategory(connection);
                // UpdateCategory(connection);
                // DeleteCategory(connection);
                // ListCategories(connection);
                // GetCategory(connection);
                // ExecuteProcedure(connection);
                // ExecuteReadProcedure(connection);
                // ExecuteScalar(connection);
                // ReadView(connection);
                // OneToOne(connection);
                // OneToMany(connection);
                // QueryMutiple(connection);
                // SelectIn(connection);
                // Like(connection, "backend");
                 Transaction(connection);
            }
        }

        static void ListCategories(SqlConnection connection) {
            var categories = connection.Query<Category>("SELECT [Id], [Title] FROM [Category]");
            foreach (var item in categories) {
                Console.WriteLine($"{item.Id} - {item.Title}");
            }
        }

        static void GetCategory(SqlConnection connection) {
            var category = connection
                .QueryFirstOrDefault<Category>(
                    "SELECT TOP 1 [Id], [Title] FROM [Category] WHERE [Id]=@id",
                    new {
                        id = "af3407aa-11ae-4621-a2ef-2028b85507c4"
                    });
            Console.WriteLine($"{category.Id} - {category.Title}");

        }

        static void CreateCategory(SqlConnection connection) {
            var category = new Category();
            category.Id = Guid.NewGuid();
            category.Title = "Amazon AWS";
            category.Url = "amazon";
            category.Description = "Categoria destinada a serviços do AWS";
            category.Order = 8;
            category.Summary = "AWS Cloud";
            category.Featured = false;

            var insertSql = @"INSERT INTO
                    [Category]
                VALUES(
                    @Id,
                    @Title,
                    @Url,
                    @Summary,
                    @Order,
                    @Description,
                    @Featured)";

            var rows = connection.Execute(insertSql, new {
                category.Id,
                category.Title,
                category.Url,
                category.Summary,
                category.Order,
                category.Description,
                category.Featured
            });
            Console.WriteLine($"{rows} linhas inseridas");
        }

        static void UpdateCategory(SqlConnection connection) {
            var updateQuery = "UPDATE [Category] SET [Title]=@title WHERE [Id]=@id";
            var rows = connection.Execute(updateQuery, new {
                id = new Guid("af3407aa-11ae-4621-a2ef-2028b85507c4"),
                title = "Frontend 2021"
            });

            Console.WriteLine($"{rows} registros atualizadas");
        }

        static void DeleteCategory(SqlConnection connection) {
            var deleteQuery = "DELETE [Category] WHERE [Id]=@id";
            var rows = connection.Execute(deleteQuery, new {
                id = new Guid("ea8059a2-e679-4e74-99b5-e4f0b310fe6f"),
            });

            Console.WriteLine($"{rows} registros excluídos");
        }

        static void CreateManyCategory(SqlConnection connection) {
            var category = new Category();
            category.Id = Guid.NewGuid();
            category.Title = "Amazon AWS";
            category.Url = "amazon";
            category.Description = "Categoria destinada a serviços do AWS";
            category.Order = 8;
            category.Summary = "AWS Cloud";
            category.Featured = false;

            var category2 = new Category();
            category2.Id = Guid.NewGuid();
            category2.Title = "Categoria Nova";
            category2.Url = "categoria-nova";
            category2.Description = "Categoria nova";
            category2.Order = 9;
            category2.Summary = "Categoria";
            category2.Featured = true;

            var insertSql = @"INSERT INTO
                    [Category]
                VALUES(
                    @Id,
                    @Title,
                    @Url,
                    @Summary,
                    @Order,
                    @Description,
                    @Featured)";

            var rows = connection.Execute(insertSql, new[]{
                new
                {
                    category.Id,
                    category.Title,
                    category.Url,
                    category.Summary,
                    category.Order,
                    category.Description,
                    category.Featured
                },
                new
                {
                    category2.Id,
                    category2.Title,
                    category2.Url,
                    category2.Summary,
                    category2.Order,
                    category2.Description,
                    category2.Featured
                }
            });
            Console.WriteLine($"{rows} linhas inseridas");
        }

        static void ExecuteProcedure(SqlConnection connection) {
            var procedure = "[spDeleteStudent]";
            var pars = new { StudentId = "6bd552ea-7187-4bae-abb6-54e8f8b9f530" };
            var affectedRows = connection.Execute(
                procedure,
                pars,
                commandType: CommandType.StoredProcedure);

            Console.WriteLine($"{affectedRows} linhas afetadas");
        }

        static void ExecuteReadProcedure(SqlConnection connection) {
            var procedure = "[spGetCoursesByCategory]";
            var pars = new { CategoryId = "09ce0b7b-cfca-497b-92c0-3290ad9d5142" };
            var courses = connection.Query(
                procedure,
                pars,
                commandType: CommandType.StoredProcedure);

            foreach (var item in courses) {
                Console.WriteLine(item.Title);
            }
        }

        static void ExecuteScalar(SqlConnection connection) {
            var category = new Category();
            category.Title = "Amazon AWS";
            category.Url = "amazon";
            category.Description = "Categoria destinada a serviços do AWS";
            category.Order = 8;
            category.Summary = "AWS Cloud";
            category.Featured = false;

            var insertSql = @"
                INSERT INTO
                    [Category]
                OUTPUT inserted.[Id]
                VALUES(
                    NEWID(),
                    @Title,
                    @Url,
                    @Summary,
                    @Order,
                    @Description,
                    @Featured)
                ";

            var id = connection.ExecuteScalar<Guid>(insertSql, new {
                category.Title,
                category.Url,
                category.Summary,
                category.Order,
                category.Description,
                category.Featured
            });
            Console.WriteLine($"A categoria inserida foi: {id}");
        }

        static void ReadView(SqlConnection connection) {
            var sql = "SELECT * FROM [vwCourses]";
            var courses = connection.Query(sql);

            foreach (var item in courses) {
                Console.WriteLine($"{item.Id} - {item.Title}");
            }
        }

        static void OneToOne(SqlConnection connection) {
            var sql = @"
                SELECT
                    *
                FROM
                    [CareerItem]
                INNER JOIN
                    [Course] ON [CareerItem].[CourseId] = [Course].[Id]";

            var items = connection.Query<CareerItem, Course, CareerItem>(
                sql,
                (careerItem, course) => {
                    careerItem.Course = course;
                    return careerItem;
                }, splitOn: "Id");

            foreach (var item in items) {
                Console.WriteLine($"{item.Title} - Curso: {item.Course.Title}");
            }
        }

        static void OneToMany(SqlConnection connection) {
            var sql = @"
                SELECT
                    [Career].[Id],
                    [Career].[Title],
                    [CareerItem].[CareerId],
                    [CareerItem].[Title]
                FROM
                    [Career]
                INNER JOIN
                    [CareerItem] ON [CareerItem].[CareerId] = [Career].[Id]
                ORDER BY
                    [Career].[Title]";

            var careers = new List<Career>();
            var items = connection.Query<Career, CareerItem, Career>(
                sql,
                (career, item) => {
                    var car = careers.Where(x => x.Id == career.Id).FirstOrDefault();
                    if (car == null) {
                        car = career;
                        car.Items.Add(item);
                        careers.Add(car);
                    } else {
                        car.Items.Add(item);
                    }

                    return career;
                }, splitOn: "CareerId");

            foreach (var career in careers) {
                Console.WriteLine($"{career.Title}");
                foreach (var item in career.Items) {
                    Console.WriteLine($" - {item.Title}");
                }
            }
        }

        static void QueryMutiple(SqlConnection connection) {
            var query = "SELECT * FROM [Category]; SELECT * FROM [Course]";

            using (var multi = connection.QueryMultiple(query)) {
                var categories = multi.Read<Category>();
                var courses = multi.Read<Course>();

                foreach (var item in categories) {
                    Console.WriteLine(item.Title);
                }

                foreach (var item in courses) {
                    Console.WriteLine(item.Title);
                }
            }
        }

        static void SelectIn(SqlConnection connection) {
            var query = @"select * from Career where [Id] IN @Id";

            var items = connection.Query<Career>(query, new {
                Id = new[]{
                    "4327ac7e-963b-4893-9f31-9a3b28a4e72b",
                    "e6730d1c-6870-4df3-ae68-438624e04c72"
                }
            });

            foreach (var item in items) {
                Console.WriteLine(item.Title);
            }

        }

        static void Like(SqlConnection connection, string term) {
            var query = @"SELECT * FROM [Course] WHERE [Title] LIKE @exp";

            var items = connection.Query<Course>(query, new {
                exp = $"%{term}%"
            });

            foreach (var item in items) {
                Console.WriteLine(item.Title);
            }
        }

        static void Transaction(SqlConnection connection) {
            var category = new Category();
            category.Id = Guid.NewGuid();
            category.Title = "Minha categoria que não";
            category.Url = "amazon";
            category.Description = "Categoria destinada a serviços do AWS";
            category.Order = 8;
            category.Summary = "AWS Cloud";
            category.Featured = false;

            var insertSql = @"INSERT INTO
                    [Category]
                VALUES(
                    @Id,
                    @Title,
                    @Url,
                    @Summary,
                    @Order,
                    @Description,
                    @Featured)";

            connection.Open();
            using (var transaction = connection.BeginTransaction()) {
                var rows = connection.Execute(insertSql, new {
                    category.Id,
                    category.Title,
                    category.Url,
                    category.Summary,
                    category.Order,
                    category.Description,
                    category.Featured
                }, transaction);

                transaction.Commit();
                // transaction.Rollback();

                Console.WriteLine($"{rows} linhas inseridas");
            }
        }
    }
}

```

---

ESTAMOS NO MODULO 4 DO CURSO VOU DEIXAR O CAMINHO DO PROJETO REALIZADO AQUI E TODA DOCUMENTAÇÃO

[https://www.notion.so/SQL-CRUD-0cfadf2d4b6e4b9a80a34bff624de25b?pvs=4](https://www.notion.so/SQL-CRUD-vers-o-beta-0cfadf2d4b6e4b9a80a34bff624de25b?pvs=21)
