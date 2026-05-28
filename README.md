##Relembrando o que é necessário 
Antes de tudo, precisamos dizer ao Windows que você tem permissão para rodar comandos novos.

Abra o PowerShell no seu computador.
Cole o comando abaixo e aperte Enter:
```Set-ExecutionPolicy RemoteSigned -Scope CurrentUser```

Baixe e instale o nvm-setup.exe encontrado em https://github.com/coreybutler/nvm-windows/releases
Rode
```nvm install latest```

Assim que terminar, ative a versão instalada digitando (substitua pelo número da versão que apareceu na tela, por exemplo):
```nvm use 25.9.0```

Instalação do Firecrawl:
```npx -y firecrawl-cli@latest init --all -k [coloque sua key]```
