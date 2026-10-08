# NotebookLM-SpringSecurity

### 1\. Conceitos Fundamentais

* **Autenticação vs. Autorização**: Autenticar é dizer quem você é. Autorizar é definir o que você pode acessar[1].
* **Status HTTP Cruciais**:
  * **401 Unauthorized**: O sistema não te conhece (falha de autenticação)[1][2].
  * **403 Forbidden**: O sistema te conhece, mas você não tem permissão para aquele recurso[1][2].
* **Stateless**: Em APIs REST, não guardamos estado de sessão no servidor. Cada requisição deve ser validada individualmente[3].

### 2\. Configuração e Dependências

* Adicione o `spring-boot-starter-security` no `pom.xml`[4].
* Por padrão, o Spring gera uma senha no log e exige **Basic Auth** (usuário: `user`)[5].
* Para JWT, utilize as dependências do **JJWT** (`api`, `impl`, `jackson`)[6].
* **Segurança da Chave**: A chave secreta do JWT deve ter no mínimo **256 bits** (344 caracteres aleatórios são recomendados) para evitar exceções de "chave fraca"[7][8].

### 3\. Implementação das Entidades

* **Roles (Papéis)**: Devem implementar `GrantedAuthority`[9].
* **Usuário (Ex: Alunos)**: Deve implementar `UserDetails`[10].
* **Métodos Obrigatórios**: Você precisará sobrescrever `getAuthorities()`, `getPassword()` e `getUsername()` (que pode ser o e-mail, desde que seja um identificador único)[11][12].

### 4\. Fluxo do JWT (Checklist Prático)

1. **TokenProvider**: Classe responsável por **gerar, validar e extrair** informações (como o subject/e-mail) do token[13][14].
2. **Filtro customizado**: Crie um filtro que estende `OncePerRequestFilter`[15].
  * Ele deve extrair o header `Authorization`[16].
  * Validar se começa com o prefixo **Bearer** [16][17].
  * Se válido, configurar o usuário no **SecurityContextHolder**[18].
3. **UserDetailsService**: Implemente esta interface para ensinar o Spring a buscar o usuário no seu banco de dados pelo "username"[19][20].

### 5\. Classe de Configuração (`SecurityFilterChain`)

* Anote com `@Configuration` e `@EnableWebSecurity`[21].
* **Desabilite o CSRF** (comum em APIs stateless)[3].
* Defina as rotas públicas (geralmente `/login` e `/register`) com `.permitAll()`[22][23].
* **Ordem do Filtro**: Sempre adicione seu filtro JWT **antes** do `UsernamePasswordAuthenticationFilter`[23][24].

### 6\. Dicas de Ouro e "Pegadinhas"

* **Criptografia de Senha**: **Nunca** salve senhas em texto puro. Use o bean `BCryptPasswordEncoder` para salvar o hash no banco[25].
* **Fetch Eager**: Na entidade de usuário, o relacionamento com as Roles deve ser **FetchType.EAGER**. Se for `LAZY`, o filtro de segurança pode lançar uma exceção ao tentar validar permissões fora de uma transação[28].
* **Prefixo "ROLE\_"**: Ao usar `.hasRole("ADMIN")`, o Spring procura por **ROLE\_ADMIN** no banco. Se quiser usar o nome exato do banco, use `.hasAuthority()`[29][30].
* **Segurança por Método**: Ative com `@EnableMethodSecurity`. Isso permite usar `@PreAuthorize` diretamente nos Controllers para validações complexas, como checar se o ID do recurso pertence ao usuário logado: `authentication.principal.id == #id`[31][32].
* **Tratamento de Erros**: Crie handlers para `AccessDeniedException` para garantir que o cliente receba um 403 amigável em vez de um erro 500 genérico


### Mapa mental
<img width="5619" height="8936" alt="NotebookLM Mind Map" src="https://github.com/user-attachments/assets/d3095476-a4e9-4875-b6d8-a06f58532692" />
