<?php
$host = "localhost";
$user = "root";
$password = "";
$dbname = "pn_36351425_hecate";

try {
    $pdo = new PDO("mysql:host=$host;dbname=$dbname;charset=utf8", $user, $password);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    
    $sql_criar_tabela = "CREATE TABLE IF NOT EXISTS usuarios (
        id INT AUTO_INCREMENT PRIMARY KEY,
        nick VARCHAR(100) NOT NULL UNIQUE,
        senha VARCHAR(255) NOT NULL,
        data_criacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )";
    $pdo->exec($sql_criar_tabela);
    
} catch(PDOException $e) {
    $pdo = null;
}

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    
    if (!(isset($_POST['botao1']) or isset($_POST['botao2']))){
        header("location: inicio.php");
        exit();
    }

    if(isset($_POST['botao1'])){
        $senha = md5($_POST['codigo']);
        
        if (!is_dir('usuarios')) {
            mkdir('usuarios', 0755, true);
        }
        
        $arq = fopen("usuarios/" . $_POST['nick'], "w");
        fwrite($arq, $senha);
        fclose($arq);
        
        $arquivo_txt = "usuarios/usuarios.txt";
        $dados_usuario = "Usuário: " . $_POST['nick'] . " | Senha: " . $senha . " | Data: " . date('d/m/Y H:i:s') . PHP_EOL;
        file_put_contents($arquivo_txt, $dados_usuario, FILE_APPEND | LOCK_EX);
        
        if ($pdo) {
            try {
                $sql = "INSERT INTO usuarios (nick, senha) VALUES (?, ?)";
                $stmt = $pdo->prepare($sql);
                $stmt->execute([$_POST['nick'], password_hash($_POST['codigo'], PASSWORD_DEFAULT)]);
            } catch(PDOException $e) {
            }
        }
        
        $mensagem = "Usuário registrado com sucesso!";
    }

    if (isset($_POST['botao2'])){
        if ($pdo) {
            try {
                $sql = "SELECT * FROM usuarios WHERE nick = ?";
                $stmt = $pdo->prepare($sql);
                $stmt->execute([$_POST['nick']]);
                $usuario = $stmt->fetch();
                
                if ($usuario && password_verify($_POST['codigo'], $usuario['senha'])) {
                    $arq = fopen("logado", "w");
                    fwrite($arq, $_POST['nick']);
                    fclose($arq);
                    
                    $arquivo_log = "usuarios/logs.txt";
                    $log_entrada = "LOGIN BEM-SUCEDIDO (BANCO) - Usuário: " . $_POST['nick'] . " | Data: " . date('d/m/Y H:i:s') . PHP_EOL;
                    file_put_contents($arquivo_log, $log_entrada, FILE_APPEND | LOCK_EX);
                    
                    header("location: dashboard.html");
                    exit();
                }
            } catch(PDOException $e) {
            }
        }
        
        if (file_exists("usuarios/" . $_POST['nick'])) {
            $arq = fopen("usuarios/" . $_POST['nick'], "r");
            $senha1 = fgets($arq, 10000);
            fclose($arq);
            $senha = md5($_POST['codigo']);
            
            if($senha == $senha1) {
                $arq = fopen("logado", "w");
                fwrite($arq, $_POST['nick']);
                fclose($arq);
                
                $arquivo_log = "usuarios/logs.txt";
                $log_entrada = "LOGIN BEM-SUCEDIDO - Usuário: " . $_POST['nick'] . " | Data: " . date('d/m/Y H:i:s') . PHP_EOL;
                file_put_contents($arquivo_log, $log_entrada, FILE_APPEND | LOCK_EX);
                
                header("location: dashboard.html");
                exit();
            } else {
                $arquivo_log = "usuarios/logs.txt";
                $log_entrada = "TENTATIVA DE LOGIN FALHA - Usuário: " . $_POST['nick'] . " | Data: " . date('d/m/Y H:i:s') . PHP_EOL;
                file_put_contents($arquivo_log, $log_entrada, FILE_APPEND | LOCK_EX);
                
                $erro = "Senha incorreta!";
            }
        } else {
            $erro = "Usuário não encontrado!";
        }
    }
}
?>
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hecate - Plataforma de Jogos</title>
    <link rel="stylesheet" href="style.css">
    <style>
        .mensagem {
            background: #4CAF50;
            color: white;
            padding: 10px;
            margin: 10px 0;
            border-radius: 4px;
            text-align: center;
        }
        .erro {
            background: #f44336;
            color: white;
            padding: 10px;
            margin: 10px 0;
            border-radius: 4px;
            text-align: center;
        }
    </style>
</head>
<body>
    <div class="container">

        <header class="header">
            <div class="logo">Hecate</div>
            <nav class="nav-menu">
                <a href="#" class="nav-item">Inicio</a>
                <a href="#" class="nav-item">Catálogo</a>
                <a href="#" class="nav-item">Downloads</a>
                <a href="#" class="nav-item">Ajustes</a>
            </nav>
        </header>

        <section class="login-section">
            <h2 class="section-title">Login</h2>
            
            <?php
            if (isset($mensagem)) {
                echo '<div class="mensagem">' . $mensagem . '</div>';
            }
            if (isset($erro)) {
                echo '<div class="erro">' . $erro . '</div>';
            }
            ?>
            
            <form method="post" class="login-form">
                <div class="form-group">
                    <input type="text" name="nick" class="form-input" placeholder="Nome de usuário" required value="<?php echo isset($_POST['nick']) ? htmlspecialchars($_POST['nick']) : ''; ?>">
                </div>
                <div class="form-group">
                    <input type="password" name="codigo" class="form-input" placeholder="Senha" required>
                </div>
                <div class="form-group">
                    <button type="submit" name="botao1" class="btn">Registrar</button>
                    <button type="submit" name="botao2" class="btn btn-secondary" style="margin-left: 10px;">Entrar</button>
                </div>
            </form>
        </section>

        <div class="main-content">

            <div class="content-left">
                <section class="content-section">
                    <h2 class="content-title">Destaques</h2>
                    <div class="game-card">
                        <h3 class="game-title">Hollow Knight: Silksong</h3>
                        <p class="game-description">
                            Desculpa um reino vasto e amaldiçoado em Hollow Knight: Silksong! 
                            Explore, lute e sobreviva enquanto você ascende ao pico.
                        </p>
                    </div>
                </section>

                <section class="content-section">
                    <h2 class="content-title">Populares</h2>
                    <div class="game-card">
                        <h3 class="game-title">V3.6.8 "Lumer"</h3>
                        <p class="game-description">Alguns downloads em andamento</p>
                    </div>
                </section>
            </div>

            <aside class="sidebar">
                <h3 class="content-title">BIBLIOTECA</h3>
                <ul class="category-list">
                    <li class="category-item">Buscar</li>
                    <li class="category-item">Populares</li>
                    <li class="category-item">Mais baixados da semana</li>
                    <li class="category-item">Pra platinar</li>
                    <li class="category-item">Surpreenda-me</li>
                </ul>
            </aside>
        </div>

        <footer class="status-bar">
            Sistema operacional: V3.6.8 "Lumer" | Status: Online
        </footer>
    </div>
</body>
</html>
