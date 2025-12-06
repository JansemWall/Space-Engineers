using VRage.Game.GUI.TextPanel; // Necessário para usar Sprites

// --- CONFIGURAÇÕES --- //
const double MIN_GELO_KG = 1000;
const float ENERGIA_CRITICA = 0.20f;
const float O2_CRITICO = 0.20f;
const string NOME_LCD = "LCD_Energia";

// Cores do Tema
Color CorFundo = new Color(20, 20, 20);
Color CorBarraFundo = new Color(60, 60, 60);
Color CorTexto = new Color(200, 200, 200);

// --- OBJETOS --- //
IMyGasGenerator _o2Gen;
IMyPowerProducer _hydroEngine;
IMyTextPanel _lcd;
List<IMyBatteryBlock> _baterias = new List<IMyBatteryBlock>();
List<IMyPowerProducer> _produtores = new List<IMyPowerProducer>();
List<IMyGasTank> _tanquesH2 = new List<IMyGasTank>();
List<IMyGasTank> _tanquesO2 = new List<IMyGasTank>();

// Helper de Renderização
SurfaceRenderer _renderer;

public Program()
{
    Runtime.UpdateFrequency = UpdateFrequency.Update100;
    InicializarBlocos();
    
    // Inicializa o renderizador se o LCD existir
    if (_lcd != null)
    {
        _renderer = new SurfaceRenderer(_lcd);
    }
}

public void Main(string argument, UpdateType updateSource)
{
    // 1. Coleta (Igual ao anterior, simplificado para focar no visual)
    double geloAtual = _o2Gen != null ? (double)_o2Gen.GetInventory(0).CurrentMass : 0;
    float nivelEnergia = CalcularNivelEnergia();
    double nivelH2 = ObterMediaTanques(_tanquesH2);
    double nivelO2 = ObterMediaTanques(_tanquesO2);
    
    // 2. Lógica (Simplificada aqui)
    bool motorLigado = false;
    if (_hydroEngine != null)
    {
        motorLigado = geloAtual > MIN_GELO_KG && nivelEnergia < 0.9f && nivelH2 > 0.1f;
        _hydroEngine.Enabled = motorLigado;
    }

    // 3. Renderização (Onde a mágica acontece)
    if (_renderer != null)
    {
        _renderer.BeginFrame();

        _renderer.DrawHeader("SISTEMA DE SUPORTE");

        // Desenha barras com lógica de cor automática
        _renderer.DrawProgressBar("Energia Baterias", nivelEnergia, ENERGIA_CRITICA);
        _renderer.DrawProgressBar("Tanques H2", (float)nivelH2, 0.1f);
        _renderer.DrawProgressBar("Tanques O2", (float)nivelO2, O2_CRITICO);
        
        // Exibe status do Gelo e Motor
        _renderer.DrawInfoBox($"Gelo: {geloAtual:N0} kg", motorLigado ? "MOTOR: ON" : "MOTOR: OFF", motorLigado ? Color.Green : Color.Gray);

        _renderer.EndFrame();
    }
}

// --- CLASSE DE RENDERIZAÇÃO (FRONTEND) --- //
public class SurfaceRenderer
{
    IMyTextSurface _surface;
    RectangleF _viewport;
    Vector2 _posAtual;
    float _padding = 10f;

    public SurfaceRenderer(IMyTextSurface surface)
    {
        _surface = surface;
        // Configura o LCD para modo Sprite (Script)
        _surface.ContentType = ContentType.SCRIPT;
        _surface.Script = ""; 
        _surface.ScriptBackgroundColor = new Color(20, 20, 20); // Fundo escuro
        
        // Calcula a área desenhável
        Vector2 textureSize = _surface.TextureSize;
        float margin = 20f;
        _viewport = new RectangleF(
            (textureSize - _surface.SurfaceSize) / 2f + margin, 
            _surface.SurfaceSize - (margin * 2)
        );
    }

    MySpriteDrawFrame _frame;

    public void BeginFrame()
    {
        _frame = _surface.DrawFrame();
        _posAtual = _viewport.Position; // Reseta o cursor de desenho para o topo
    }

    public void EndFrame()
    {
        _frame.Dispose(); // Envia os comandos para a GPU do jogo
    }

    public void DrawHeader(string title)
    {
        // Desenha texto centralizado
        var sprite = MySprite.CreateText(title, "Debug", Color.White, 1.2f, TextAlignment.CENTER);
        sprite.Position = new Vector2(_viewport.Center.X, _posAtual.Y);
        _frame.Add(sprite);
        _posAtual.Y += 40f; // Pula linha
    }

    public void DrawProgressBar(string label, float percentage, float criticalThreshold)
    {
        // Configuração visual
        float barHeight = 30f;
        float barWidth = _viewport.Width;
        Color barColor = percentage < criticalThreshold ? Color.Red : (percentage < 0.5f ? Color.Yellow : Color.Green);

        // 1. Texto do Label
        var labelSprite = MySprite.CreateText($"{label}: {percentage:P0}", "Debug", Color.White, 0.8f, TextAlignment.LEFT);
        labelSprite.Position = _posAtual;
        _frame.Add(labelSprite);
        _posAtual.Y += 25f;

        // 2. Fundo da Barra (Cinza escuro)
        var bgSprite = new MySprite(SpriteType.TEXTURE, "SquareSimple", size: new Vector2(barWidth, barHeight), color: new Color(50, 50, 50));
        bgSprite.Position = new Vector2(_viewport.Center.X, _posAtual.Y + barHeight/2);
        _frame.Add(bgSprite);

        // 3. Preenchimento da Barra (Colorido)
        float fillWidth = barWidth * MathHelper.Clamp(percentage, 0f, 1f);
        if (fillWidth > 0)
        {
            var fillSprite = new MySprite(SpriteType.TEXTURE, "SquareSimple", size: new Vector2(fillWidth, barHeight), color: barColor);
            // Posição X = Esquerda da tela + Metade da largura da barra atual
            fillSprite.Position = new Vector2(_viewport.X + (fillWidth / 2), _posAtual.Y + barHeight/2);
            _frame.Add(fillSprite);
        }

        _posAtual.Y += barHeight + _padding; // Pula para o próximo elemento
    }

    public void DrawInfoBox(string textLeft, string textRight, Color statusColor)
    {
        _posAtual.Y += 10f;
        var spriteLeft = MySprite.CreateText(textLeft, "Debug", Color.White, 0.8f, TextAlignment.LEFT);
        spriteLeft.Position = new Vector2(_viewport.X, _posAtual.Y);
        _frame.Add(spriteLeft);

        var spriteRight = MySprite.CreateText(textRight, "Debug", statusColor, 0.8f, TextAlignment.RIGHT);
        spriteRight.Position = new Vector2(_viewport.X + _viewport.Width, _posAtual.Y);
        _frame.Add(spriteRight);
    }
}

// --- MÉTODOS DE SUPORTE --- //
void InicializarBlocos() {
    _o2Gen = GridTerminalSystem.GetBlockWithName("O2/H2 Generator") as IMyGasGenerator;
    _hydroEngine = GridTerminalSystem.GetBlockWithName("Hydrogen Engine") as IMyPowerProducer;
    _lcd = GridTerminalSystem.GetBlockWithName(NOME_LCD) as IMyTextPanel;
    GridTerminalSystem.GetBlocksOfType(_baterias, b => b.IsSameConstructAs(Me));
    GridTerminalSystem.GetBlocksOfType(_tanquesH2, t => t.IsSameConstructAs(Me) && t.BlockDefinition.SubtypeName.Contains("HydrogenTank"));
    GridTerminalSystem.GetBlocksOfType(_tanquesO2, t => t.IsSameConstructAs(Me) && t.BlockDefinition.SubtypeName.Contains("OxygenTank"));
}
float CalcularNivelEnergia() {
    if (_baterias.Count == 0) return 0f;
    return _baterias.Sum(b => b.CurrentStoredPower) / _baterias.Sum(b => b.MaxStoredPower);
}
double ObterMediaTanques(List<IMyGasTank> l) {
    if (l.Count == 0) return 0;
    return l.Average(t => (double)t.FilledRatio);
}
