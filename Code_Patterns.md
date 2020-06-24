# Notas de padronização de código
#
#
#
##### Nomenclaturas
#
`Todos os nomes utilizados no código serão em inglês.`
1. Variáveis
   1.1. Variáveis com letra inicial minúscula; e
   1.2. Quando usar nomes com várias palavras, usar CamelCase, porém mantendo a regra 1;

    *Exemplo:*
    ```
    GameObject playerMaster;
    float speed;
    int jumpCount = 3;
    ```
    
2. **Métodos**
2.1. Nome de métodos com a inicial maiúscula, separação maiuscula e em ingles

    *Exemplo:*
    ```
    Jump();
    DeathFunction(); 
    Shoot(GameObject shootablePrefab);
    TakeDamage(float damageDealt, int hitCounts);
    ```

#
##### Modificadores de acesso de variáveis (public, private, protected)
#
- **`public`**: 
    Usadas principalmente por variáveis que serão modificadas diversas vezes no inspetor e/ou via outro código.

    *NOTA:* Caso ela não vá ser modificada no inspetor e vá ser modificada por outras classes, use o comando `[HideInInspector]` para ocupar menos espaço no inspetor.


- **`private` ou `sem marcação` de acesso (apenas a declaração sem public ou private)**: 
    Variáveis que podem ser acessadas *apenas dentro da própria classe* (não podem ser acessadas de outras classes).
    
    *NOTA*: Considerar usar o comando `[SerializeField]` em cima delas para que sejam acessadas via inspetor, ao invés de usar variáveis públicas.

- **`public static`**: 
    Usar para variáveis únicas, que podem ser acessadas diretamente via `SuaClasse.nomeVariavel` sem precisar instanciar ou referenciar a classe anteriormente.
    
    *Exemplo*: GameObject do player, nível do player, missões etc.

#
##### Objetos únicos na cena (player, boss, miniboss, NPCs únicos etc.)
#
- Criar uma variável pública e estática para referenciá-lo diretamente de qualquer lugar; e 
- Não usar tag ou layer para fazer verificação de colisão.

    *Exemplo:*
    ```
    //na sua classe você faz assim:
    public static GameObject objectX;
    ```
    
- Usar `Awake()` para salvar a referência do `objectX` citado no ponto anterior, pois alguns códigos que rodam no `Start()` de outros objetos podem tentar acessar o objeto e este ainda não haver sido referenciado (o que chamamos de race condition - haveria uma condição de corrida para que o objeto fosse referenciado no *Start()* de sua classe antes do *Start()* da outra classe tentar usá-lo). Lembrar que o *Awake()* sempre é executado antes do *Start()*.

    *Exemplo*:
    ```
    void Awake()
    {
    objectX = gameObject;
    }
    ```
    *Outro exemplo:*
    ```
    //Em outro código
    Transform myTargetTransform;
    
    //void Start() tentando acessar o transform
    void Start()
    {
         myTargetTransform = CodeWhereTheObjectXAreLocated.objectX.transform;
    }
    
    //Exemplo em colisão
    void OnCollisionEnter2D(Collision2D whoIHit)
    {
       if(whoIHit.collider.gameObject == CodeWhereTheObjectXAreLocated.objectX)
       {
    	print("bati no objeto X");
       }
    }
    ```

#
##### Objetos repetidos na cena (inimigos, projéteis, itens coletáveis etc.) 
#
- Verificar colisão por tag (mais usada para verificações generalizadas); e
- Caso seja necessário especificar o objeto ou a classe, usar `GetComponent<T>()`.

    *Exemplo:*
    ```
    //verificação simples
    void OnCollisionEnter2D(Collision2D whoIHit)
    {
       if(whoIHit.gameObject.CompareTag("StringDaTagDoAlvo"))
       {
    	print("bati no objeto com a tag x");
       }
    }
    ```
    *Outro exemplo:*
    ```
    //verificação de inimigos
    void OnCollisionEnter2D(Collision2D whoIHit)
    {
       //verifica se bateu inimigos
       if(whoIHit.gameObject.CompareTag("inimigos"))
       {
          //verifica em qual inimigo bateu, verificando pelo script
          if(whoIHit.gameObject.GetComponent<InimigoA>() == true)
          {
                print("bati no inimigo a");
          }
          else if(whoIHit.gameObject.GetComponent<InimigoB>() == true)
          {
                print("bati no inimigo b");
          }
       }
    }
    ```

#
##### Objetos de cena estáticos (cenário, portas, escadas, pontos de transição)
#
* Utilizar verficação via layer.

    *Exemplo:*
    ```
    //verificação simples
    void OnCollisionEnter2D(Collision2D whoIHit)
    {
       if(whoIHit.gameObject.layer == LayerMask.NameToLayer("NameOfTheLayer"))
       {
    	    print("bati no objeto com a layer x");
       }
    }
    ```

#
##### Comandos de inspetor que podem ser úteis
#
* `[Header("string")]`
Adiciona um texto fixo em destaque no inspetor. Útil para separar e organizar as variáveis.

* `[HideInInspector]`
Remove variáveis públicas do inspetor (esconde elas), sem retirar o acesso público da variável.

* `[SerializeField]`
Mostra variáveis privadas no inspetor, sem modificar o acesso privado delas.

* `[Range(numero minimo,numero maximo)]`
Adiciona um *slider* no inspetor para modificar o valor de uma variável numérica.

* `[MinMaxSlider(numero minimo, numero maximo)]`
Possui a mesma funcão do Range citado acima, porém usando basicamente um `Vector2` onde a propriedade `x` do vetor representa um valor mínimo e a `y` um máximo (ambos selecionados no inspetor, porém com uma GUI um pouco diferente). Muito usado para, por exemplo, casos onde se deseja usar aleatoriedades, como no `Random.Range(valorMinimo, valorMaximo)`, onde se "sorteia" um valor entre os dois valores passados como parâmetro.

    *Exemplo:* 
    ```
    [MinMaxSlider(1f, 5f)] //Pode-se usar float ou int nos parâmetros, porém ambos precisam ser do mesmo tipo
    Vector2 time;
    
    IEnumerator WaitBeforeSpawning() 
    {
        float randomTime = Random.Range(time.x, time.y);
        yield return new WaitForSeconds(randomTime);
        Spawn();
    }
    ```

* `[RequireComponent(typeof(T))]`
Usado na linha acima do nome de declaração da classe para forçar a criação de um componente/script do tipo `T` dentro da hierarquia de componentes de um GameObject, caso este componente ainda não exista no momento em que se insere a classe, que possui esta linha, dentro do ojeto.