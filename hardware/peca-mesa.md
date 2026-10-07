# Peça de mesa (QR Code + NFC)

## Função

Levar o cliente até `https://<dominio>/m/{slug_publico}`. A peça **não guarda estado**: tudo é resolvido no servidor pelo slug, então ela pode ser revogada e reemitida sem alterar a mesa.

## Especificação inicial

| Item | Valor |
|---|---|
| Material | PETG (resiste a limpeza com álcool e a calor moderado) |
| Formato | Display de mesa triangular ou plaquinha com base, entre 60 e 80 mm de largura |
| Tag NFC | NTAG213 (144 bytes, suficiente para a URL) ou NTAG215, adesiva, 25 mm |
| Embutimento | Pausar a impressão na camada logo acima do rebaixo da tag. Deixar entre 0,6 e 1,0 mm de material sobre a tag |
| QR Code | Adesivo de vinil laminado ou bicolor por troca de filamento. Lado mínimo de 30 mm. Correção de erro nível M ou superior |
| Texto | "Aproxime o celular ou leia o QR para dividir a conta" + rótulo da mesa |
| Restrições | Nenhum metal a menos de 10 mm da tag. Testar leitura em mesa de inox e de pedra |

## Gravação da tag

- Gravar um registro NDEF do tipo URI com a URL completa.
- Depois de validar, aplicar o lock de escrita, para impedir regravação maliciosa.
- Registrar o UID da tag em `PECA_MESA.nfc_uid`.

## Testes de aceitação

- [ ] Leitura NFC em pelo menos 5 modelos de celular (Android de entrada e iPhone XS ou mais recente).
- [ ] Leitura do QR com pouca luz e a 40 cm de distância.
- [ ] Peça intacta após 50 limpezas com álcool 70%.
- [ ] Peça revogada leva a uma página de erro genérica.
