# Como atualizar o Stats personalizado

Toda segunda-feira, por volta das 06:17 (horário de Brasília), o GitHub pega a versão mais nova do Stats oficial, reaplica as nossas mudanças (o widget de disco com o SSD e o disco Ugreen) e monta um app novo. Você instala quando quiser. Nada é instalado sozinho.

## 1. Baixar a versão mais nova

Abra o Terminal e cole:

```bash
sha=$(gh api repos/byvict/stats/commits/custom --jq .sha)
rm -rf ~/Downloads/stats-build
gh run download -R byvict/stats -n "Stats-$sha" -D ~/Downloads/stats-build
ditto -x -k ~/Downloads/stats-build/Stats.zip ~/Downloads/stats-build
```

Baixe sempre pelo `gh`, nunca pelo navegador. O navegador marca o arquivo como "baixado da internet" e o macOS se recusa a abrir o app.

Se aparecer `no artifact matches any of the names or patterns provided`, a versão da semana ainda está sendo montada ou falhou. Veja a seção 4.

## 2. Trocar o app

```bash
osascript -e 'quit app "Stats"'
rm -rf ~/Downloads/Stats-anterior.app
mv /Applications/Stats.app ~/Downloads/Stats-anterior.app
ditto ~/Downloads/stats-build/Stats.app /Applications/Stats.app
open /Applications/Stats.app
```

A versão anterior fica guardada em `~/Downloads/Stats-anterior.app`. Se a nova der problema, feche o Stats e coloque a anterior de volta em `/Applications` com o nome `Stats.app`.

O macOS pode pedir de novo alguma permissão que você já tinha dado ao Stats. É normal, porque cada versão montada pelo GitHub tem uma assinatura nova.

## 3. Conferir que a atualização automática continua desligada

```bash
defaults read eu.exelban.Stats update-interval
```

Tem que aparecer `Never`. Se aparecer outra coisa, ou um erro, abra o Stats, vá nas configurações gerais e escolha "Never" na opção de verificar atualizações. Sem isso, o Stats baixa sozinho a versão oficial por cima da nossa e o widget volta a mostrar um disco só. Pelo mesmo motivo, nunca aceite uma atualização oferecida pelo próprio Stats.

## 4. Se chegar um e-mail de falha do GitHub

As execuções ficam em https://github.com/byvict/stats/actions/workflows/fork-sync.yaml.

- **"Rebase conflict".** O Stats oficial mexeu nos mesmos trechos que as nossas mudanças. Peça a um Claude para "resolver o conflito do fork do Stats seguindo o FORK.md". Enquanto isso, o app que você já tem continua funcionando.
- **"Push rejected".** O Stats oficial mudou os próprios arquivos de automação e falta a chave de deploy descrita no FORK.md. Peça a um Claude para configurá-la.
- **Qualquer outra falha, como "not acquired by Runner".** Faltou máquina no GitHub. Abra a execução e clique em "Re-run failed jobs", ou espere a segunda seguinte, que tenta de novo sozinha.

Para pedir uma versão nova na hora, abra o link acima, clique em "Run workflow" e depois no botão verde "Run workflow".
