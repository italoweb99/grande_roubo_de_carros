
# Grande Roubo de Carros


## 🎮 Instruções de Jogabilidade

### Controle do Personagem (A pé)
* **W / S / A / D**: Movimentação do Herói (Frente, Trás, Rotação Esquerda, Rotação Direita).
* **F (Próximo à porta)**: Entrar no Veículo.

### Controle do Carro (Dirigindo)
* **W / S (Seta Cima / Baixo)**: Acelerar e dar Marcha Ré.
* **A / D (Seta Esquerda / Direita)**: Esterçar as rodas (Direção).
* **Espaço**: Ativar Freio de Mão (Acende as luzes de freio traseiras e gera frenagem).
* **E**: Ligar / Desligar os Faróis dianteiros.
* **F**: Sair do Veículo (O personagem reaparece ao lado da porta com segurança).

---

## 📺 Gameplay em Vídeo

Confira a mecânica de transição de controle, a alternância de câmeras e o comportamento do áudio dinâmico em funcionamento:

*(Para exibir o vídeo, salve a sua gravação com o nome `gameplay.mp4` na pasta raiz do seu repositório do GitHub)*
<video src = "https://github.com/user-attachments/assets/fa8fff45-d8ac-4415-a402-543c97c7520a" width="100%" controls></video>
)

---

## 📸 Capturas de Tela do Jogo

Abaixo estão os registros visuais das três principais perspectivas de câmera e comportamentos físicos do projeto:

### 1. Visão Geral em Terceira Pessoa (Modo Pedestre)
<img width="704" height="390" alt="Captura de tela 2026-06-14 231035" src="https://github.com/user-attachments/assets/a7129188-286f-4bea-9fe8-1acbd014f66a" /> 

*Legenda: Perspectiva padrão focada na aproximação do herói até a porta do motorista.*

### 2. Direção e Pilotagem Ativa (Modo Veículo)
<img width="702" height="397" alt="Captura de tela 2026-06-14 231201" src="https://github.com/user-attachments/assets/8c613f05-508e-43f4-8107-694cbafc05ea" />

*Legenda: Assunção total dos controles de torque das rodas e câmera do veículo ativa.*

### 3. Menu
<img width="696" height="393" alt="Captura de tela 2026-06-14 231242" src="https://github.com/user-attachments/assets/c73783a4-f04a-478e-8ed1-41f4c1e0ebd8" />


---

## 🛠️ Funcionalidades Principais & Códigos

### Adicionado som de motor no carro
Adiconado som ao carro com modulação de *pitch* conforne a aceleração para que o jogo fique mais imersivo

```
csharp

using UnityEngine;

public class SomCarro : MonoBehaviour
{
    [Header("Componentes")]
    public AudioSource AudioMotor;
    private Rigidbody rb; 

    [Header("Configurações do Som")]
    public float PitchMinimo = 0.8f;  
    public float PitchMaximo = 2.5f;  
    public float VelocidadeMaximaDoCarro = 30f; 

    private float velocidadeAtual;

    void Start()
    {
        if (AudioMotor == null)
        {
            AudioMotor = GetComponent<AudioSource>();
        }
        rb = GetComponent<Rigidbody>();
    }

    void Update()
    {
        if (rb != null)
        {
            // Mede a velocidade física real do veículo em m/s
            velocidadeAtual = rb.linearVelocity.magnitude; 
        }

        // Mapeia a velocidade proporcionalmente dentro dos limites de Pitch definidos
        float proporcaoVelocidade = velocidadeAtual / VelocidadeMaximaDoCarro;
        AudioMotor.pitch = Mathf.Lerp(PitchMinimo, PitchMaximo, proporcaoVelocidade);
    }
}

```
### Adicionado interação carro e personagem
Foi adicionado a interação entre personagem e carro para que fosse possivel dirigir o carro

#### No script do personagem (`ControlePersonagem.cs`)

```csharp
// --- VARIÁVEIS DE INTERAÇÃO ---
[Header("Sistema de Entrada (Distância)")]
public Carro carroAlvo; // Referência direta ao script do carro
public float distanciaParaInteragir = 2.0f; // Distância limite para conseguir entrar

void Update()
{
    if (carroAlvo == null) return;

    // 1. MEDIÇÃO: Calcula a distância matemática exata entre o herói e a porta do carro
    float distanciaAtePorta = Vector3.Distance(transform.position, carroAlvo.pontoDeSaida.position);

    // 2. VERIFICAÇÃO: Se estiver perto o suficiente e pressionar F
    if (distanciaAtePorta <= distanciaParaInteragir && Input.GetKeyDown(KeyCode.F))
    {
        EntrarNoCarro();
        return; 
    }
}

void EntrarNoCarro()
{
    // 3. SEGURANÇA: Zera a velocidade física para o corpo do player não empurrar o carro
    if (rb != null)
    {
        rb.linearVelocity = Vector3.zero; 
        rb.angularVelocity = Vector3.zero;
    }

    // 4. CHAMADA: Avisa o script do carro que o jogador entrou
    carroAlvo.EntrarNoVeiculo(this);
    
    // 5. TRANSIÇÃO: Desativa o objeto do personagem na cena (ele "some")
    gameObject.SetActive(false); 
}
```
#### No script do carro (`Carro.cs`)
```csharp
// --- VARIÁVEIS DE INTERAÇÃO ---
[Header("Configurações do Sistema de Entrada")]
public Camera cameraDoCarro; // Câmera focada no veículo
public Transform pontoDeSaida; // Objeto vazio posicionado na porta do motorista
private bool podeDirigir = false; // Bloqueio principal dos comandos de aceleração
private ControlePersonagem personajeNoCarro; // Guarda quem é o motorista atual
private AudioListener listenerCarro; // Guarda o receptor de áudio da câmera do carro

void Update()
{
    // Se não tiver ninguém dirigindo, o carro ignora totalmente o teclado
    if (!podeDirigir) return; 

    // 3. SAÍDA: Se o jogador já estiver dentro e pressionar F de novo, ele desembarca
    if (Input.GetKeyDown(KeyCode.F) && personagemNoCarro != null)
    {
        SairDoCarro();
    }
}

// 1. RECEBIMENTO (Chamado pelo Personagem)
public void EntrarNoVeiculo(ControlePersonagem player)
{
    personagemNoCarro = player; // Guarda a referência de quem entrou
    podeDirigir = true;         // Libera as funções do FixedUpdate (acelerar/virar)

    // Ativa os olhos e ouvidos do carro
    if (cameraDoCarro != null) cameraDoCarro.gameObject.SetActive(true);
    if (listenerCarro != null) listenerCarro.enabled = true;

    // Liga o motor
    if (audioMotor != null) audioMotor.Play();
}

// 2. DESEMBARQUE (Chamado pelo próprio Update do Carro)
void SairDoCarro()
{
    podeDirigir = false; // Bloqueia os comandos do carro imediatamente

    // Desliga os olhos e ouvidos do carro antes do jogador voltar (evita erro de 2 Listeners)
    if (listenerCarro != null) listenerCarro.enabled = false;
    if (cameraDoCarro != null) cameraDoCarro.gameObject.SetActive(false);

    // Reposiciona o personagem exatamente onde a porta (pontoDeSaida) está
    personagemNoCarro.transform.position = pontoDeSaida.position;
    personagemNoCarro.transform.rotation = pontoDeSaida.rotation;
    
    // Reativa o herói na cena (ele reaparece na porta)
    personagemNoCarro.gameObject.SetActive(true);

    // Desliga o motor
    if (audioMotor != null) audioMotor.Stop();

    personagemNoCarro = null; // Limpa a vaga do motorista
}
```
