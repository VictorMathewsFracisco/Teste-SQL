## Instruções:

- Cada questão possui uma situação específica seguida por uma solicitação.
- Escreva consultas SQL para atender às solicitações.
- Utilize o banco de dados de exemplo fornecido (se necessário) ou assuma a existência de tabelas relevantes.
- Forneça as consultas SQL completas para cada questão.
- Avalie a eficiência das consultas em termos de desempenho sempre que possível.
- Envie os resultados do teste, de preferência, em um repositório público.
- Este teste tem a finalidade de avaliar suas habilidades em vários níveis, portanto, realize o máximo de questões possíveis, não se preocupe se não conseguir realizar alguma questão.


## Banco de Dados de Exemplo:

Considere o seguinte esquema de banco de dados para referência:

```
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    DepartmentID INT,
    Salary DECIMAL(10, 2),
    HireDate DATE
);

CREATE TABLE Departments (
    DepartmentID INT PRIMARY KEY,
    DepartmentName VARCHAR(50)
);

CREATE TABLE Projects (
    ProjectID INT PRIMARY KEY,
    ProjectName VARCHAR(100),
    DepartmentID INT,
    StartDate DATE,
    EndDate DATE
);

CREATE TABLE Assignments (
    AssignmentID INT PRIMARY KEY,
    EmployeeID INT,
    ProjectID INT,
    AssignmentDate DATE,
    HoursWorked DECIMAL(5, 2)
);

CREATE TABLE Reviews (
    ReviewID INT PRIMARY KEY,
    EmployeeID INT,
    ReviewDate DATE,
    Rating INT
);
```

## Questões:

### 1. Recuperação de Dados
Você é solicitado a recuperar o nome completo (FirstName e LastName) de todos os funcionários que foram contratados no ano de 2023.

Consulta:

SELECT
	FirstName AS PrimeiroNome,
	LastName AS Sobrenome
FROM Employees
WHERE HireDate >= '2023-01-01' 
	AND HireDate <= '2023-12-31'
;

### 2. Cálculos e Agregações
Calcule a média de salário por departamento. Liste o nome do departamento e a média de salário, classificados em ordem decrescente pela média de salário.

Consulta:

SELECT
	d.DepartmentName AS Departamento,
	AVG(e.Salary) AS MediaSalarial
FROM Employees e
INNER JOIN Departments d ON d.DepartmentID = e.DepartmentID
GROUP BY d.DepartmentName
ORDER BY MediaSalarial DESC
;

### 3. Desafio de Desempenho
Escreva uma consulta eficiente para encontrar o funcionário mais antigo (em termos de data de contratação) em cada departamento. A consulta deve retornar o DepartmentID, o EmployeeID e a data de contratação do funcionário mais antigo em cada departamento.

Consulta:

SELECT
	d.DepartmentID AS Departamento,
	e.EmployeeID AS IDFuncionario,
	MIN(e.HireDate) AS ContratacaoMaisAntiga
FROM Employees e
INNER JOIN Departments d ON d.DepartmentID = e.DepartmentID
GROUP BY d.DepartmentID
;

### 4. Otimização de Consulta
O departamento de "Recursos Humanos" tem um grande número de funcionários. Escreva uma consulta eficiente para contar o número total de funcionários neste departamento.

Consulta:

SELECT
	COUNT(*) AS TotalFuncionarios
FROM Employees
WHERE DepartmentID IN (
						SELECT DepartmentID
						FROM Departments
						WHERE DepartmentName = 'Recursos Humanos'
					   )
;

### 5. Desempenho de Atualização
Uma atualização em massa deve ser realizada em todas as linhas da tabela Employees, aumentando o salário de todos os funcionários em 10%. Escreva uma consulta que execute essa atualização de forma eficiente, minimizando o impacto no desempenho do sistema.

Consulta:

UPDATE Employees
SET Salary = Salary * 1.1
;

### 6. Recuperação de Dados Complexa
Recupere o nome completo (FirstName e LastName) dos funcionários, juntamente com o nome do departamento em que trabalham e o nome do projeto ao qual estão atualmente atribuídos. Além disso, inclua a data de início do projeto e a quantidade total de horas trabalhadas pelo funcionário no projeto até o momento.

Consulta:

SELECT
	e.FirstName AS PrimeiroNome,
	e.LastName AS Sobrenome,
	d.DepartmentName AS Departamento,
	p.ProjectName AS Projeto,
	p.StartDate AS DataInicio,
	SUM(a.HoursWorked) AS HorasTrabalhadas
FROM Employees e
INNER JOIN Departments d ON d.DepartmentID = e.DepartmentID
INNER JOIN Assignments a ON a.EmployeeID = e.EmployeeID
INNER JOIN Projects p ON p.ProjectID = a.ProjectID
GROUP BY e.FirstName, e.LastName, d.DepartmentName,	p.ProjectName, p.StartDate
;

### 7. Agregação e Filtragem
Calcule a média de classificação (Rating) das revisões dos funcionários em cada departamento. Liste o nome do departamento e a média de classificação, incluindo apenas os departamentos onde a média de classificação seja maior que 3.

Consulta:

SELECT
	d.DepartmentName AS Departamento,
	AVG(r.Rating) AS MediaClassificacao
FROM Employees e
INNER JOIN Departments d ON d.DepartmentID = e.DepartmentID
INNER JOIN Reviews r ON r.EmployeeID = e.EmployeeID
GROUP BY d.DepartmentName
HAVING AVG(r.Rating) > 3
;

### 8. Junção de Dados e Ordenação
Recupere o nome completo (FirstName e LastName) dos funcionários que foram atribuídos a projetos com uma duração superior a 6 meses. Ordene os resultados pelo nome do projeto em ordem alfabética.

Consulta:

SELECT
	e.FirstName AS PrimeiroNome,
	e.LastName AS Sobrenome,
	p.ProjectName AS Projeto
FROM Employees e
INNER JOIN Assignments a ON a.EmployeeID = e.EmployeeID
INNER JOIN Projects p ON p.ProjectID = a.ProjectID
WHERE DATEDIFF(p.EndDate, p.StartDate) > 180
ORDER BY p.ProjectName
;

### 9. Análise de Desempenho
Identifique os funcionários que trabalharam em pelo menos três projetos diferentes nos últimos 12 meses. Liste o nome completo desses funcionários e a contagem de projetos em que estiveram envolvidos.

Consulta:

SELECT
	e.FirstName AS PrimeiroNome,
	e.LastName AS Sobrenome,
	COUNT(DISTINCT(p.ProjectID)) AS QuantidadeProjetos
FROM Employees e
INNER JOIN Assignments a ON a.EmployeeID = e.EmployeeID
INNER JOIN Projects p ON p.ProjectID = a.ProjectID
WHERE a.AssignmentDate >= DATE_ADD(CURDATE(), INTERVAL -12 MONTH)
GROUP BY e.EmployeeID
HAVING COUNT(DISTINCT(p.ProjectID)) >= 3
;

### 10. Consulta Complexa de Atualização
Uma nova política salarial foi implementada, aumentando o salário de todos os funcionários em 5%, exceto aqueles que estão atribuídos a projetos com uma duração superior a 9 meses. Atualize os salários dos funcionários de acordo com essa política.

Consulta:

UPDATE Employees e
SET Salary = Salary * 1.05
WHERE NOT EXISTS (
					SELECT 1
					FROM Assignments a
					INNER JOIN Projects p ON p.ProjectID = a.ProjectID
					WHERE a.EmployeeID = e.EmployeeID
						AND DATEDIFF(p.EndDate, p.StartDate) > 270
				 )
;