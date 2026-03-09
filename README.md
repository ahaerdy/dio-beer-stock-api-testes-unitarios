# 🍺 Beer Stock API — Gerenciamento de Estoque de Cerveja

> Projeto desenvolvido durante a **Live Coding da Digital Innovation One (DIO)**
> Tema: *Desenvolvimento de Testes Unitários para validar uma API REST de Gerenciamento de Estoques de Cerveja*

---

## 📋 Índice

- [Sobre o Projeto](#sobre-o-projeto)
- [Arquitetura e Estrutura](#arquitetura-e-estrutura)
- [Tecnologias Utilizadas](#tecnologias-utilizadas)
- [Pré-requisitos](#pré-requisitos)
- [Como Executar](#como-executar)
- [Endpoints da API](#endpoints-da-api)
- [Testes Unitários](#testes-unitários)
  - [Pirâmide de Testes](#pirâmide-de-testes)
  - [Frameworks de Teste](#frameworks-de-teste)
  - [Testando a Camada Controller](#testando-a-camada-controller)
  - [Testando a Camada Service](#testando-a-camada-service)
  - [TDD: Incremento e Decremento de Estoque](#tdd-incremento-e-decremento-de-estoque)
- [Estrutura dos Pacotes](#estrutura-dos-pacotes)
- [Explicação Didática das Classes](#explicação-didática-das-classes)
- [Referências](#referências)

---

## 📖 Sobre o Projeto

Esta API REST foi construída com **Spring Boot** para gerenciar o estoque de cervejas. O foco principal do projeto é a **escrita de testes unitários** de qualidade, cobrindo as camadas de *Controller* e *Service* da aplicação.

Durante a aula, foram abordados:

- A **pirâmide de testes** e a importância de cada nível
- **Testes unitários** com JUnit 5, Mockito e Hamcrest
- A prática de **TDD (Test-Driven Development)** aplicada às funcionalidades de incremento e decremento de estoque

---

## 🏗️ Arquitetura e Estrutura

O projeto segue a arquitetura em camadas padrão do Spring Boot:

```
Request HTTP
     │
     ▼
┌─────────────┐
│  Controller │  ← Recebe requisições, valida entrada, retorna respostas HTTP
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Service   │  ← Contém a lógica de negócio
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Repository  │  ← Acessa o banco de dados (H2 em memória)
└─────────────┘
```

O padrão **DTO (Data Transfer Object)** é utilizado para desacoplar a entidade do banco de dados do contrato público da API. A conversão entre `Beer` (entidade) e `BeerDTO` é feita pelo **MapStruct**.

---

## 🛠️ Tecnologias Utilizadas

| Tecnologia | Versão | Papel no Projeto |
|---|---|---|
| Java | 14+ | Linguagem principal |
| Spring Boot | 2.x | Framework principal |
| Spring Web MVC | — | Criação dos endpoints REST |
| Spring Data JPA | — | Persistência de dados |
| H2 Database | — | Banco de dados em memória (dev/test) |
| Lombok | — | Redução de boilerplate (getters, setters, builders) |
| MapStruct | — | Mapeamento automático entre entidade e DTO |
| JUnit 5 | — | Framework de testes unitários |
| Mockito | — | Criação de mocks e stubs para isolamento de testes |
| Hamcrest | — | Matchers expressivos para asserções |
| Maven | 3.6.3+ | Gerenciamento de dependências e build |

---

## ✅ Pré-requisitos

- **Java 14** ou superior
- **Maven 3.6.3** ou superior
- **Git** instalado
- Uma IDE como IntelliJ IDEA, Eclipse ou VS Code

> 💡 Dica: Use o [SDKMan!](https://sdkman.io/) para gerenciar versões do Java e Maven facilmente.

---

## ▶️ Como Executar

### Clonar o repositório

```bash
git clone https://github.com/SEU_USUARIO/SEU_REPOSITORIO.git
cd SEU_REPOSITORIO
```

### Executar a aplicação

```bash
mvn spring-boot:run
```

A API estará disponível em: `http://localhost:8080/api/v1/beers`

### Executar os testes

```bash
mvn clean test
```

---

## 🌐 Endpoints da API

Base URL: `http://localhost:8080/api/v1/beers`

| Método HTTP | Endpoint | Descrição |
|---|---|---|
| `POST` | `/` | Cadastra uma nova cerveja |
| `GET` | `/` | Lista todas as cervejas |
| `GET` | `/{name}` | Busca uma cerveja pelo nome |
| `DELETE` | `/{id}` | Remove uma cerveja pelo ID |
| `PATCH` | `/{id}/increment` | Incrementa a quantidade em estoque |
| `PATCH` | `/{id}/decrement` | Decrementa a quantidade em estoque |

### Exemplo de corpo da requisição (POST)

```json
{
  "name": "Brahma",
  "brand": "Ambev",
  "maxStock": 50,
  "quantity": 10,
  "type": "LAGER"
}
```

---

## 🧪 Testes Unitários

### Pirâmide de Testes

A pirâmide de testes define a proporção ideal de cada tipo de teste:

```
        /\
       /  \
      / E2E\      ← Poucos: lentos, caros, frágeis
     /──────\
    / Integ. \    ← Médio: validam integração entre módulos
   /──────────\
  /  Unitários \  ← Muitos: rápidos, baratos, isolados
 /──────────────\
```

**Testes unitários** são a base da pirâmide — devem ser rápidos e testar **uma unidade de código de forma isolada**, sem dependências externas reais (banco de dados, rede, etc.). Para isso usamos **mocks**.

---

### Frameworks de Teste

#### JUnit 5
O JUnit é o framework principal de execução de testes. Anotações mais usadas neste projeto:

```java
@ExtendWith(MockitoExtension.class) // integra Mockito ao JUnit 5
@Test                               // marca o método como um teste
@BeforeEach                         // executa antes de cada teste
```

#### Mockito
O Mockito permite criar **objetos falsos (mocks)** que simulam o comportamento de dependências reais, permitindo testar uma classe em isolamento.

```java
@Mock
BeerRepository beerRepository;         // cria um mock do repositório

when(beerRepository.findByName("X"))   // define o comportamento do mock
    .thenReturn(Optional.of(beer));

verify(beerRepository, times(1))       // verifica se o método foi chamado
    .findByName("X");
```

#### Hamcrest
O Hamcrest fornece **matchers** (verificadores) que tornam as asserções mais expressivas e legíveis:

```java
assertThat(beerDTO.getName(), is(equalTo("Brahma")));
assertThat(beers, is(not(empty())));
assertThat(beers.size(), is(1));
```

---

### Testando a Camada Controller

Os testes do `BeerController` utilizam o `MockMvc` — uma ferramenta do Spring que simula requisições HTTP sem subir um servidor real.

```java
@ExtendWith(MockitoExtension.class)
class BeerControllerTest {

    private static final String BEER_API_URL_PATH = "/api/v1/beers";

    @Mock
    private BeerService beerService;

    @InjectMocks
    private BeerController beerController;

    private MockMvc mockMvc;

    @BeforeEach
    void setUp() {
        mockMvc = MockMvcBuilders.standaloneSetup(beerController).build();
    }

    @Test
    void whenPOSTIsCalledThenABeerIsCreated() throws Exception {
        // dado
        BeerDTO beerDTO = BeerDTOBuilder.builder().build().toBeerDTO();

        // quando
        when(beerService.createBeer(beerDTO)).thenReturn(beerDTO);

        // então
        mockMvc.perform(post(BEER_API_URL_PATH)
                .contentType(MediaType.APPLICATION_JSON)
                .content(asJsonString(beerDTO)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.name", is(beerDTO.getName())));
    }
}
```

**O que esse teste valida:**
- O endpoint `POST /api/v1/beers` retorna status `201 Created`
- O corpo da resposta contém o campo `name` correto
- O `BeerService.createBeer()` foi chamado com os dados corretos

---

### Testando a Camada Service

Os testes do `BeerService` isolam a lógica de negócio, mockando o repositório e o mapper:

```java
@ExtendWith(MockitoExtension.class)
class BeerServiceTest {

    @Mock
    private BeerRepository beerRepository;

    @Mock
    private BeerMapper beerMapper;

    @InjectMocks
    private BeerService beerService;

    @Test
    void whenBeerInformedThenItShouldBeCreated() throws BeerAlreadyRegisteredException {
        // dado
        BeerDTO expectedBeerDTO = BeerDTOBuilder.builder().build().toBeerDTO();
        Beer expectedSavedBeer = beerMapper.toModel(expectedBeerDTO);

        // quando
        when(beerRepository.findByName(expectedBeerDTO.getName())).thenReturn(Optional.empty());
        when(beerRepository.save(expectedSavedBeer)).thenReturn(expectedSavedBeer);

        // então
        BeerDTO createdBeerDTO = beerService.createBeer(expectedBeerDTO);

        assertThat(createdBeerDTO.getId(), is(equalTo(expectedBeerDTO.getId())));
        assertThat(createdBeerDTO.getName(), is(equalTo(expectedBeerDTO.getName())));
    }
}
```

---

### TDD: Incremento e Decremento de Estoque

**TDD (Test-Driven Development)** é uma técnica onde o **teste é escrito antes do código de produção**, seguindo o ciclo:

```
🔴 RED    → Escreve o teste (falha porque o código não existe)
🟢 GREEN  → Implementa o mínimo para o teste passar
🔵 REFACTOR → Melhora o código sem quebrar os testes
```

#### Teste de Incremento (TDD)

```java
@Test
void whenIncrementIsCalledThenIncrementBeerStock() throws Exception {
    // dado
    BeerDTO expectedBeerDTO = BeerDTOBuilder.builder().build().toBeerDTO();
    expectedBeerDTO.setQuantity(expectedBeerDTO.getQuantity() + 10);

    // quando
    when(beerService.increment(VALID_BEER_ID, 10)).thenReturn(expectedBeerDTO);

    // então
    mockMvc.perform(patch(BEER_API_URL_PATH + "/" + VALID_BEER_ID + BEER_API_SUBPATH_INCREMENT_URL)
            .contentType(MediaType.APPLICATION_JSON)
            .content(asJsonString(new QuantityDTO(10))))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name", is(expectedBeerDTO.getName())))
            .andExpect(jsonPath("$.quantity", is(expectedBeerDTO.getQuantity())));
}

@Test
void whenIncrementIsGreaterThanMaxThenThrowException() throws Exception {
    // quando o incremento excede o estoque máximo, deve lançar exceção
    when(beerService.increment(VALID_BEER_ID, 999))
        .thenThrow(BeerStockExceededException.class);

    mockMvc.perform(patch(BEER_API_URL_PATH + "/" + VALID_BEER_ID + BEER_API_SUBPATH_INCREMENT_URL)
            .contentType(MediaType.APPLICATION_JSON)
            .content(asJsonString(new QuantityDTO(999))))
            .andExpect(status().isBadRequest());
}
```

**Regra de negócio validada:** A quantidade em estoque nunca pode ultrapassar o valor de `maxStock` da cerveja.

---

## 📁 Estrutura dos Pacotes

```
src/
├── main/
│   └── java/one/digitalinnovation/beerstock/
│       ├── controller/
│       │   └── BeerController.java       # Camada REST: endpoints HTTP
│       ├── dto/
│       │   ├── BeerDTO.java              # Objeto de transferência de dados
│       │   └── QuantityDTO.java          # DTO para operações de estoque
│       ├── entity/
│       │   └── Beer.java                 # Entidade JPA (tabela no banco)
│       ├── enums/
│       │   └── BeerType.java             # Enum: tipos de cerveja (LAGER, ALE...)
│       ├── exception/
│       │   ├── BeerAlreadyRegisteredException.java
│       │   ├── BeerNotFoundException.java
│       │   └── BeerStockExceededException.java
│       ├── mapper/
│       │   └── BeerMapper.java           # Conversão Beer ↔ BeerDTO (MapStruct)
│       ├── repository/
│       │   └── BeerRepository.java       # Interface JPA (acesso ao banco)
│       └── service/
│           └── BeerService.java          # Lógica de negócio
└── test/
    └── java/one/digitalinnovation/beerstock/
        ├── builder/
        │   └── BeerDTOBuilder.java        # Builder para criar DTOs de teste
        ├── controller/
        │   └── BeerControllerTest.java    # Testes da camada Controller
        └── service/
            └── BeerServiceTest.java       # Testes da camada Service
```

---

## 📚 Explicação Didática das Classes

### `Beer.java` (Entidade)
Representa a tabela `BEER` no banco de dados. Anotações JPA definem o mapeamento objeto-relacional.

```java
@Entity
@Data                    // Lombok: gera getters, setters, equals, hashCode
@Builder                 // Lombok: habilita o padrão Builder
@NoArgsConstructor
@AllArgsConstructor
public class Beer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String name;

    @Column(nullable = false)
    private String brand;

    @Column(nullable = false)
    private int max;        // quantidade máxima em estoque

    @Column(nullable = false)
    private int quantity;   // quantidade atual

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private BeerType type;
}
```

### `BeerDTO.java` (Data Transfer Object)
Representa os dados que trafegam na API (entrada e saída). Usa **Bean Validation** para garantir integridade dos dados recebidos.

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class BeerDTO {
    private Long id;

    @NotNull
    @Size(min = 1, max = 200)
    private String name;

    @NotNull
    @Size(min = 1, max = 200)
    private String brand;

    @NotNull
    @Max(500)
    private Integer max;

    @NotNull
    @Max(300)
    private Integer quantity;

    @NotNull
    private BeerType type;
}
```

### `BeerMapper.java` (MapStruct)
Interface que o MapStruct implementa automaticamente em tempo de compilação, convertendo entre `Beer` e `BeerDTO`:

```java
@Mapper(componentModel = "spring")
public interface BeerMapper {
    Beer toModel(BeerDTO beerDTO);
    BeerDTO toDTO(Beer beer);
}
```

### `BeerRepository.java` (Spring Data JPA)
Extende `JpaRepository` e ganha **CRUD completo automaticamente**. O método `findByName` é gerado pelo Spring a partir do nome do método:

```java
@Repository
public interface BeerRepository extends JpaRepository<Beer, Long> {
    Optional<Beer> findByName(String name);
}
```

### `BeerService.java` (Lógica de Negócio)
Contém todas as regras de negócio, incluindo validações de estoque:

```java
@Service
@AllArgsConstructor
public class BeerService {

    private final BeerRepository beerRepository;
    private final BeerMapper beerMapper;

    public BeerDTO createBeer(BeerDTO beerDTO) throws BeerAlreadyRegisteredException {
        verifyIfIsAlreadyRegistered(beerDTO.getName());
        Beer beer = beerMapper.toModel(beerDTO);
        Beer savedBeer = beerRepository.save(beer);
        return beerMapper.toDTO(savedBeer);
    }

    public BeerDTO increment(Long id, int quantityToIncrement)
            throws BeerNotFoundException, BeerStockExceededException {
        Beer beerToIncrementStock = verifyIfExists(id);
        int quantityAfterIncrement = quantityToIncrement + beerToIncrementStock.getQuantity();
        if (quantityAfterIncrement <= beerToIncrementStock.getMax()) {
            beerToIncrementStock.setQuantity(quantityAfterIncrement);
            Beer incrementedBeerStock = beerRepository.save(beerToIncrementStock);
            return beerMapper.toDTO(incrementedBeerStock);
        }
        throw new BeerStockExceededException(id, quantityToIncrement);
    }
}
```

### `BeerDTOBuilder.java` (Builder de Testes)
Padrão **Builder** utilizado nos testes para criar objetos com dados consistentes sem repetir código:

```java
@Builder
public class BeerDTOBuilder {

    @Builder.Default
    private Long id = 1L;

    @Builder.Default
    private String name = "Brahma";

    @Builder.Default
    private String brand = "Ambev";

    @Builder.Default
    private int max = 50;

    @Builder.Default
    private int quantity = 10;

    @Builder.Default
    private BeerType type = BeerType.LAGER;

    public BeerDTO toBeerDTO() {
        return new BeerDTO(id, name, brand, max, quantity, type);
    }
}
```

---

## Referências

- [SDKMan! — Gerenciamento do Java e Maven](https://sdkman.io/)
- [Site oficial do Spring Boot](https://spring.io/)
- [Documentação JUnit 5](https://junit.org/junit5/docs/current/user-guide/)
- [Site oficial Mockito](https://site.mockito.org/)
- [Site oficial Hamcrest](http://hamcrest.org/JavaHamcrest/)
- [Testes com Spring Boot — Baeldung](https://www.baeldung.com/spring-boot-testing)
- [Padrão arquitetural REST](https://restfulapi.net/)
- [Pirâmide de Testes — Martin Fowler](https://martinfowler.com/articles/practical-test-pyramid.html)

---

## 👨‍💻 Autor

Projeto baseado no conteúdo da **Digital Innovation One (DIO)**
Professor: Rodrigo Peleias
Fork e documentação por: **Arthur Haerdy**
