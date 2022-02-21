package application; //Demo: atualizar dados

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

import db.DB;

public class Program {

	public static void main(String[] args) {

		//Um programa para atualizar o salario de um vendedor.
		Connection conn = null;
		PreparedStatement st = null;
		try {
			//Primeira coisa eu vou me conectar com o banco de dados.
			conn = DB.getConnection();
			st = conn.prepareStatement(
					//aqui dentro eu vou colocar o comando para atualizar um registro do banco de dados.
					"UPDATE seller "
					/*
					 * o comando set que é pra atualizar um valor, qual valor? o sálario, ai vamos dizer que eu quero 
					 * acrementar o salario, pegar o que ja tinha e acrementar mais, ai eu faço assim?
					 * SET BaseSalary = BaseSalary + ?, esse interrogação vai ser o valor
					 */
					+ "SET BaseSalary = BaseSalary + ? "
					/*
					 * agora eu vou concatenar aqui a palavra WHERE pra eu ter uma restrição na minha atualização, nunca faça
					 * uma atualização sem por uma restrição, se não você vai atualizar todo mundo, o que é um dessastre no
					 * banco de dados. Em baixo do WHERE eu vou colocar a minha restrição.
					 */
					+ "WHERE "
					/*
					 * a minha restrição vai ser o seguinte: eu vou atualizar os salarios de todos os funcionario de um certo
					 * departamento, então eu vou falar tecnicamente o seguinte "ATUALIZA os vendedores"
					 * + "MUDANDO o Salario para o Salario + um valor que eu informar" + "ONDE" + "(DepartamentId for igual
					 * a um valor que eu informar)";
					 */
					+ "(DepartmentId = ?)");
		
		//agora eu tenho que atribuir valores para os dois interrogações de cima, o valor do aumento e o codigo do departamento
	
			st.setDouble(1, 200.0); //o valor do aumento.
			st.setInt(2, 2); //o código do departamento.
			
			int rowsAffected = st.executeUpdate();
			System.out.println("Done! Rows affecteds: " + rowsAffected);
		}
		catch (SQLException e) {
			e.printStackTrace();
		}
		finally {
			DB.closeStatement(st);
			DB.closeConnection();
		}
	}
}

===========================================================================

package db;

import java.io.FileInputStream;
import java.io.IOException;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;

public class DB {
	
	private static Connection conn = null;
	/*
	 * um metodo para conectar com os bancos de dados
	 */
	public static Connection getConnection() {
		if (conn == null) { // se ainda estiver nullo, eu vou ter que fazer um codigo para conectar com o banco de dados.
			try {
				/*
				 * pra conectar com o banco a primeira coisa que vou fazer é pegar as propriedades de conexão usando 
				 * Properties props = loadProperties();, pronto, peguei aqui as propriedades do banco de dados
				 * utilizando o loadProperties() la de baixo no final do programa.
				 */
				Properties props = loadProperties();
				String url = props.getProperty("dburl");
				/*
				 * com essas propriedades em maos eu vou criar uma String URL que vai ser o meu url do banco de dados,
				 * como que eu vou pegar essa string? eu vou acessar o meu objeto props e chamar o .getProperty() e entre ()
				 * eu vou chamar o nome da propriedades que é "dburl" porque que é dburl? por que é isso que está definido
				 * no arquivo db.properties.
				 */
				conn = DriverManager.getConnection(url, props);
				/*
				 * e au pra que eu possa obter uma conexão com o meu banco de dados eu vou ter que chamar o
				 * DriverManager.getConnection(url, props); e isso é vou atribuir pra minha variavel de cima chamada conn.
				 * 
				 * então o que fazemos aqui, basicamente a gente pegou as propriedades no caso aqui a url, precisava aqui
				 * pro parametro do metodo tambem e conectamos com o nosso banco de dados utilizando o  
				 * DriverManager.getConnection(url, props);.
				 * conectar com banco de dados aqui no jdbc é instanciar um objeto do tipo Connection, que é o que a gente fez 
				 * usando esse comando aqui: DriverManager.getConnection(url, props);. Perceba que salvei esse objeto nessa
				 * variavel chamada conn, então agora a minha conexão ela está salva, a proxima vez que esse metodo
				 * getConnection for chamado, esse meu test conn == null vai falhar e ele vai simplesmente pular esse if
				 * e retornar a conn  ja existente.
				 */
			}
			catch (SQLException e) {
				throw new DbException(e.getMessage());
			}
		}
		return conn;
	}
	
	// metodo pra fechar a conexão
	public static void closeConnection() {
		if (conn != null) { // se o conn estiver estanciado eu chamo o conn.close();
			try {
				conn.close();
			}
			catch (SQLException e) {
				throw new DbException(e.getMessage());
			}
		}
	}
	/*
	 * metodo auxiliar private porque ele vai ser usado somente aqui nessa classe.
	 * esse aqui então é metodo para carregar as propiedades que estão definidas aqui no arquivo db.properties.
	 */
	private static Properties loadProperties() {
		try(FileInputStream fs = new FileInputStream("db.properties")){
			/*
			 * esse try é pra abrir os arquivos, ai eu instancio esse fs com um new FileInputStream() passando entre parenteses
			 * o parametro que é o nome do arquivo, como meu arquivo está na pasta raiz do projetopasta colocar o nome assim
			 * que vai da certo.
			 */
			Properties props = new Properties();
			/*
			 * muito bem, então a implementação desse try vai ser o seguinte, eu vou instanciar um objeto do tipo Properties
			 * ai eu vou fazer o seguinte, props.load(); recebendo como parametro o fs, esse comando load aqui do objeto
			 * properties, ele faz a leitura do arquivo properties apontado pelo meu FileInputStream fs, e vai guardar
			 * os dados dentro do objeto props.
			 */
			props.load(fs);
			// e no final aqui eu retorno o props
			return props;
			/*
			 * so que aqui essa operação pode dar algum erro, então ta faltando aqui eu tratar uma excessão.
			 */
		}
		catch(IOException e) {
			throw new DbException(e.getMessage());
		}
	}
	//vou criar um metodo pra fechar um objeto do tipo statement.
	public static void closeStatement(Statement st) {
		/*
		 * então eu vou implementar um metodo pra fechar um statement st, como vou fechar esse metodo? basicamente eu vou 
		 * fazer o seguinte, se esse st foi diferente de null, ou seja, se ele estiver instanciado, eu vou chamar o st.close()
		 * só que esse st.close() pode gerar excessão SQLExeption, então o que eu vou fazer? eu vou aceitar a correção
		 * Surround with try/catch, so que agora no caso de acontecer uma excessão, eu vou chamar o meu DbException 
		 * passando a mensagem da excessão SQLException.
		 * por que? por que ai eu vou poder ter aqui o fechamento de statement só que lançando uma excessão RuntimeException
		 * então não vai ser obrigado a ficar tratando isso toda hora com try/catch, então no programa principal, no lugar de
		 * colocar o st.close, eu vou colocar o DB.closeStatement(st);
		 * ai depois é só fazer a mesma coisa com o resultSet.
		 */
		if (st != null) {
			try {
				st.close();
			}
			catch (SQLException e) {
				throw new DbException(e.getMessage());
			}
		}
	}
	
	public static void closeResultSet(ResultSet rs) {
		if (rs != null) {
			try {
				rs.close();
			}
			catch (SQLException e) {
				throw new DbException(e.getMessage());
			}
		}
	}
}

===========================================================================

package db;

public class DbException extends RuntimeException {

	private static final long serialVersionUID = 1L;

	public DbException(String msg) {
		super(msg);
	}
}
