# Testes Stellarium-Unity

## Preparação

Seguindo o README em https://github.com/Stellarium/stellarium-unity, foram realizados os seguintes passos:

- Download do repositório em https://github.com/Stellarium/stellarium-unity;
- Criado um novo projeto Unity;
- importado o package `stellarium-unity-spout-JSONobject-U2017-3.unitypackage` para o projeto;
- Desativada a Main Camera e Directional Light;
- Adicionados os prefabs Stellarium e PFSController à scene.

Alguns erros de compilação impediam os scripts dos prefabs de serem lidos. Foi necessário corrigir esses erros.

### Erros de compilação
- Assets/Spout/Scripts/SpoutReceiver.cs dava o erro
    ```
    Assets/Spout/Scripts/SpoutReceiver.cs(150,4): error CS0592: 
    Attribute 'SerializeField' is not valid on this declaration type.
    It is only valid on 'field' declarations.
    ```
    O problema era o uso de `[SerializeField]` no método `public string sharingName`. No inicio do script já existe 
    ```csharp
    [SerializeField]
    private string _sharingName;
    ```
    Por isso foi retirado o `[SerializeField]` do método e considerado um erro dos autores.

Foram encontrados erros de assinatura nos seguintes scripts:
- Assets/stellarium-unity/StelLight.cs
    Em vez de
    ```csharp
    sunImpostorSphere = transform.Find(name: "LightImpostorSphere").gameObject;
    ```
    Deve ser
    ```csharp
    transform.Find("LightImpostorSphere").gameObject
    ```
- Assets/stellarium-unity/StreamingSkybox.cs
    Em vez de
    ```csharp
    texture = new Texture2D(width: 512, height: 512, format: TextureFormat.BGRA32, mipmap: true, linear: false);
    ```
    A [documentação](https://docs.unity3d.com/6000.2/Documentation/ScriptReference/Texture2D-ctor.html) refere que a assinatura certa é:
    ```csharp
    public Texture2D(int width, int height, TextureFormat textureFormat = TextureFormat.RGBA32, bool mipChain = true, bool linear = false);
    ```
    Neste caso fica
    ```csharp
    texture = new Texture2D(512, 512, TextureFormat.BGRA32, true, false);
    ```
### Deprecated e erros devido a substituições

Todo o código original foi comentado e deixado nos scripts para referência futura.

- Assets/stellarium-unity/StreamingSkybox.cs
    ```csharp
    uwr.chunkedTransfer = false;
    ```
    >'UnityWebRequest.chunkedTransfer' is obsolete: 'HTTP/2 and many HTTP/1.1 servers don't support this; we recommend leaving it set to false (default).'
    
    A linha foi comentada pois o default é `false`.
    
    ---
    ```csharp
    if (uwr.isNetworkError || uwr.isHttpError)
    ```
    
    > 'UnityWebRequest.isNetworkError' is obsolete: 'UnityWebRequest.isNetworkError is deprecated. Use (UnityWebRequest.result == UnityWebRequest.Result.ConnectionError) instead.'

    > 'UnityWebRequest.isHttpError' is obsolete: 'UnityWebRequest.isHttpError is deprecated. Use (UnityWebRequest.result == UnityWebRequest.Result.ProtocolError) instead.'
    
    Substituido por
    ```csharp
    if (uwr.result == UnityWebRequest.Result.ConnectionError || 
                    uwr.result == UnityWebRequest.Result.ProtocolError)
    ```

---

- Assets/Spout/Scripts/Spout.cs

    ```csharp
    _instance = GameObject.FindObjectOfType<Spout>();
    ```
    > 'Object.FindObjectOfType<T>()' is obsolete: 'Object.FindObjectOfType has been deprecated. Use Object.FindFirstObjectByType instead or if finding any instance is acceptable the faster Object.FindAnyObjectByType'

    Substituido por
    ```csharp
    _instance = GameObject.FindAnyObjectByType<Spout>();
    ```
---
- Assets/stellarium-unity/StelController.cs

    ```csharp
    uwr.chunkedTransfer = false;
    ```
    > 'UnityWebRequest.chunkedTransfer' is obsolete: 'HTTP/2 and many HTTP/1.1 servers don't support this; we recommend leaving it set to false (default).'

    Linha comentada

    ---
    ```csharp
    if (uwr.isNetworkError)
    ```
    > 'UnityWebRequest.isNetworkError' is obsolete: 'UnityWebRequest.isNetworkError is deprecated. Use (UnityWebRequest.result == UnityWebRequest.Result.ConnectionError) instead.'

    Substituido por
    ```csharp
    if (uwr.result == UnityWebRequest.Result.ConnectionError)
    ```
    ---
    ```csharp
    else if (uwr.isHttpError)
    ```
    > 'UnityWebRequest.isHttpError' is obsolete: 'UnityWebRequest.isHttpError is deprecated. Use (UnityWebRequest.result == UnityWebRequest.Result.ProtocolError) instead.'

    Substituido por
    ```csharp
    else if (uwr.result == UnityWebRequest.Result.ProtocolError)
    ```
    ---
    ```csharp
    WWW www = new WWW(url);
    yield return www;
    JSONObject json2 = new JSONObject(www.text);
    ```
    > 'WWW' is obsolete: 'Use UnityWebRequest, a fully featured replacement which is more efficient and has additional features'

    Substituido por
    ```csharp
    UnityWebRequest www = UnityWebRequest.Get(url);
    yield return www.SendWebRequest();
    JSONObject json2 = new JSONObject(www.downloadHandler.text);
    ```
    Vale para qualquer inicialização de `UnityWebRequest` e `JSONObject`.

    ---
    ```csharp
    if (www.isNetworkError || www.isHttpError)
    ```
    > 'UnityWebRequest.isNetworkError' is obsolete: 'UnityWebRequest.isNetworkError is deprecated. Use (UnityWebRequest.result == UnityWebRequest.Result.ConnectionError) instead.'

    > 'UnityWebRequest.isHttpError' is obsolete: 'UnityWebRequest.isHttpError is deprecated. Use (UnityWebRequest.result == UnityWebRequest.Result.ProtocolError) instead.'

    Substituido por
    ```csharp
    if (www.result == UnityWebRequest.Result.ConnectionError || 
                    www.result == UnityWebRequest.Result.ProtocolError)
    ```
    ---
    ```csharp
    if (www.responseHeaders.Count > 0)
    ```
    > 'UnityWebRequest' does not contain a definition for 'responseHeaders' and no accessible extension method 'responseHeaders' accepting a first argument of type 'UnityWebRequest' could be found (are you missing a using directive or an assembly reference?)
    
    ```csharp
    if (www.GetResponseHeaders().Count > 0)
    ```
    ---
    ```csharp
    foreach (KeyValuePair<string, string> entry in www.responseHeaders)
    ```
    > 'UnityWebRequest' does not contain a definition for 'responseHeaders' and no accessible extension method 'responseHeaders' accepting a first argument of type 'UnityWebRequest' could be found (are you missing a using directive or an assembly reference?)

    Substituido por
    ```csharp
    foreach (KeyValuePair<string, string> entry in www.GetResponseHeaders())
    ```

    Vale para qualquer uso de `www.responseHeaders`.

    ---

- Assets/stellarium-unity/StelSkybox.cs

    ```csharp
    WWW www = new WWW("file://" + directory + filename);
    yield return www;
    ```
    > 'WWW' is obsolete: 'Use UnityWebRequest, a fully featured replacement which is more efficient and has additional features'

    Substituido por
    ```csharp
    UnityWebRequest www = UnityWebRequest.Get("file://" + directory + filename);
    yield return www.SendWebRequest();
    ```
    Vale para qualquer inicialização de `UnityWebRequest`.

    ---

    ```csharp
    www.texture
    ```

    > 'UnityWebRequest' does not contain a definition for 'texture' and no accessible extension method 'texture' accepting a first argument of type 'UnityWebRequest' could be found (are you missing a using directive or an assembly reference?)

    Substituido por
    ```charp
    DownloadHandlerTexture.GetContent(www)
    ```
    

> NOTA: por vezes o objeto `UnityWebRequest` é instanciado como `uwr` outras como `www`, no entanto as substituições seguem a mesma lógica.

Depois destas correções os scripts ficaram disponíveis no Inspector.

### Running time errors

Como os scripts ainda usam o sistema antigo de input:
```csharp
Input.GetKey(...)
Input.GetKeyUp(...)
KeyCode...
```
Foi necessário ativar;
```
Edit -> Project Settings -> Player
-> Other Settings
-> Configuration
-> Active Input Handling
-> Both
```

Mais tarde os scripts de controle devem ser re-implementados com a nova API.

## Teste com Skybox Mode

Para gerar as imagens do Skybox é necessário correr o script no Stellarium. A melhor forma é usar a Script Console que pode ser acedida através da tecla F12.

### Problema com a exportação do ficheiro unityData.txt

O ficheiro de data, por omissão com o nome `unityData.txt`, não estava a aparecer na exportação, juntamente com os PNG. A pasta por omissão é `$HOME/Pictures/Stellatium`, mas para me certificar que os caminhos estavam corretos fiz as seguintes modificações:
1. Criei a variável de ambiente `STEL_SKYBOX_DIR` usada no script;
1. Mudei o código

    ```js
    core.output("Writing images to " + DIR);
    core.output("Writing data to " + OUTPUT_DATA);
    ```
    para
    ```js
    core.debug("Writing images to " + DIR);
    core.debug("Writing data to " + OUTPUT_DATA);
    ```
    De forma a serem visível os caminhos usados pelo script na janela de debug. 

O debug demonstrou que os caminhos estavam corretos
```console
Writing images to /Users/rui/Pictures/Stellarium/
Writing data to /Users/rui/Pictures/Stellarium//unityData.txt
```
no entanto o ficheiro de data continuava a não ser criado.

A [documentação](https://stellarium.org/doc/26.0/classStelMainScriptAPI.html#a05987daccc5ca849f23f04b763f84fa0) diz que o método `saveOutputAs` guarda o ficheiro na mesma pasta do ficheiro `output.txt`. O ficheiro da origem [src/scripting/StelScriptOutput.cpp](https://github.com/Stellarium/stellarium/blob/master/src/scripting/StelScriptOutput.cpp) denota isso na [linha 61](https://github.com/Stellarium/stellarium/blob/master/src/scripting/StelScriptOutput.cpp#L61)
```cpp
const QFileInfo outputInfo(outputFile);
const QDir dir=outputInfo.dir(); // will hold complete dirname
```
No mesmo ficheiro, na [linha 74](https://github.com/Stellarium/stellarium/blob/master/src/scripting/StelScriptOutput.cpp#L74), temos o seguinte código

```cpp
qCWarning(Scripting) << "SCRIPTING CONFIGURATION ISSUE: You are trying to save to an absolutpathname or move up in directories.";
qCWarning(Scripting) << "  To enable this, check the settings in the script console";
qCWarning(Scripting) << "  or edit config.ini and set [scriptsflag_allow_write_absolute_path=true";
```
Esta mensagem é enviada para o log do Stellarium e só pode ser acedida de duas formas:

- Executando o binário da linha de comando;
- Examinando o ficheiro de log (no caso do MacOS a localização é `~/Library/Application Support/Stellarium/log.txt`).

Assim, correndo novamente o script, e examinando o log com
```console
tail -F ~/Library/Application\ Support/Stellarium/log.txt
```
Verifica-se o output
```console
[    48.914][WARN] SCRIPTING CONFIGURATION ISSUE: You are trying to save to an absolute pathname or move up in directories.
[    48.914][WARN]   To enable this, check the settings in the script console
[    48.914][WARN]   or edit config.ini and set [scripts]/flag_allow_write_absolute_path=true
```
Nos settings da consola de scripting, verifiquei que existe um setting
```
Allow scripts to store output to arbitrary directories
```
Depois de ativar este setting o ficheiro `unityData.txt` passou a ser guardado na pasta `$HOME/Pictures/Stellatium`. Note-se também, que a pasta para o ficheiro `output.txt` é, no MacOS, `~/Library/Application Support/Stellarium/`.

### Criação da Skybox estática para teste

Neste modo não é necessária uma instância do Stellarium em execução. O script `StelController` pode ser desligado.

1. Configurei o Stellarium para gerar screenshots de 512x512 pixeis como é pedido no ficheiro `Assets/stellarium-unity/StreamingSkybox.cs`;
1. Fixei o Stellarium na data do meu nascimento. Neste caso 10-8-1970 às 23:00:00;
1. Localização em Lisboa;
1. Retirei o **Ground** e a **Atmosphere**;
1. Mantive os pontos cardeais, os nomes dos planetas e das estrelas principais;
1. Nos settings da Script Console
    - Load scripts from the User Directory by default: OFF
    - Close window when script runs: ON
    - Clear output before script runs: ON
    - Allow scripts to store screenshots to other directories: OFF
    - Allow scripts to store output to arbitrary directories: ON
    - Show screen message in WaitForKey() calls; ON
1. Executei o script a partir da consola.
1. Copiei os ficheiros gerados para `Assets/StreamingAssets/SkyBoxes/1`;
1. Ao fazer play no Unity e pressionando a tecla 1 a Skybox aparece.

### Teste de Skyboxes dinâmicas

1. Ativada a opção `Allow scripts to store screenshots to other directories` nos settings da Script console do Stellarium. Isto permite que as imagens da Skybox sejam enviadas para a diretoria configurada com a variável de ambiente `STEL_SKYBOX_DIR`; 
1. Foi criado o script `run_stellarium_skybox_live.sh` que muda a variável de ambiente `STEL_SKYBOX_DIR` para a localização `Assets/StreamingAssets/SkyBoxes/live`. É necessário ter em consideração que, em running time com uma aplicação compilada, esta localização deverá estar dentro da pasta/package da aplicação;
1. Conform instruções no script `Assets/stellarium-unity/StelController.cs`, foi gerada uma primeira Skybox no Stellarium;
1. Conform instruções no script `Assets/stellarium-unity/StelController.cs`, foi ligado o plugin **Remote Control** no Stellarium, ativado o serviço com inicialização no arranque do Stellarium e confirmada a porta 8090 como porta de serviço;
1. No Unity ativei o script `StelController`;
1. Fiz play no Unity;
1. Pressionando a tecla `F11` a Skybox é atualizada, pressionando a tecla `0` a nova Skybox é carregada no Unity.

## Spout Mode

Segundo os autores, este modo funciona apenas no Windows, pelo que não o posso testar em MacOS. De qualquer forma, não tem relevância para o conceito proposto.
