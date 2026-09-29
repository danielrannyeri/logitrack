# Evidências

Capturas de tela dos defeitos identificados durante a execução da suíte.

## Critério de registro

Uma captura por defeito, escolhida por demonstrar o comportamento incorreto sem
depender de contexto adicional — em geral o estado final, que comprova a
persistência do dado inválido no sistema.

Entre as nove capturas, uma refere-se a CT-020, caso aprovado, por evidenciar que a regra de coerência entre datas está implementada em Manutenção e ausente em Viagens, o que sustenta a análise de causa registrada em BUG-04.

Os demais casos executados estão documentados na planilha
`Casos_de_Teste_LogiTrack.xlsx`, com resultado obtido, status, testador e data
de execução.

## Arquivos

| Arquivo | Caso | Defeito |
|---|---|---|
| `CT-002_bypass-autenticacao.png` | CT-002 | BUG-01 |
| `CT-005_ano-negativo.png` | CT-005 | BUG-02 |
| `CT-014_km-negativa.png` | CT-014 | BUG-03 |
| `CT-015_datas-invertidas-sucesso.png` | CT-015 | BUG-04 |
| `CT-020_manutencao-bloqueio-datas.png` | CT-020 | BUG-04 (contraste) |
| `CT-024_dashboard-desatualizado.png` | CT-024 | BUG-05 |
| `CT-016_viagens-sobrepostas.png` | CT-016 | BUG-06 |
| `CT-017_viagem-veiculo-em-manutencao.png` | CT-017 | BUG-07 |
| `CT-022_filtro-dashboard.png` | CT-022 | BUG-08 |