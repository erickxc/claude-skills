---
name: relatorios
description: Padrões de geração de relatórios no ERP ITP. Use ao criar exports de PDF ou Excel, relatórios acadêmicos, financeiros ou de estoque. Inclui jsPDF, html2canvas e XLSX.
---

# Relatórios — ERP ITP

## Biblioteca por Formato

| Formato | Biblioteca | Uso |
|---------|-----------|-----|
| PDF | jsPDF + html2canvas | Captура HTML como imagem |
| Excel/CSV | XLSX (via CDN) | Planilhas estruturadas |
| QR Code | qrcode.react | Impressão de etiquetas |

## PDF com html2canvas + jsPDF

```tsx
import jsPDF from 'jspdf';
import html2canvas from 'html2canvas';
import { useRef } from 'react';

export function RelatorioAlunos() {
  const printRef = useRef<HTMLDivElement>(null);

  async function handleExportPDF() {
    if (!printRef.current) return;

    const canvas = await html2canvas(printRef.current, {
      scale: 2,           // Alta resolução
      useCORS: true,      // Para imagens externas
      logging: false,
    });

    const imgData = canvas.toDataURL('image/png');
    const pdf = new jsPDF({
      orientation: 'portrait',  // ou 'landscape'
      unit: 'mm',
      format: 'a4',
    });

    const pdfWidth = pdf.internal.pageSize.getWidth();
    const pdfHeight = (canvas.height * pdfWidth) / canvas.width;

    pdf.addImage(imgData, 'PNG', 0, 0, pdfWidth, pdfHeight);
    pdf.save(`relatorio-alunos-${new Date().toISOString().split('T')[0]}.pdf`);
  }

  return (
    <div>
      <Button onClick={handleExportPDF}>Exportar PDF</Button>

      {/* Área que será capturada */}
      <div ref={printRef} className="p-8 bg-white text-black">
        <h1>Relatório de Alunos — ITP</h1>
        {/* conteúdo do relatório */}
      </div>
    </div>
  );
}
```

## Excel com XLSX (CDN)

```tsx
// XLSX carregado via CDN no layout — não como npm package

declare const XLSX: any;  // tipagem global

async function handleExportExcel(dados: any[]) {
  const ws = XLSX.utils.json_to_sheet(dados);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, 'Dados');

  // Largura das colunas
  ws['!cols'] = [
    { wch: 30 },  // Nome
    { wch: 20 },  // Email
    { wch: 15 },  // Matrícula
  ];

  XLSX.writeFile(wb, `relatorio-${new Date().toISOString().split('T')[0]}.xlsx`);
}
```

## Estrutura de Relatório PDF

```tsx
// Cabeçalho padrão
<div className="flex justify-between items-center mb-6">
  <div>
    <h1 className="text-xl font-bold">Instituto Tiapretinha</h1>
    <p className="text-sm text-gray-600">Relatório gerado em {new Date().toLocaleDateString('pt-BR')}</p>
  </div>
  <img src="/logo.png" alt="ITP" className="h-12" />
</div>

// Tabela de dados
<table className="w-full border-collapse text-sm">
  <thead>
    <tr className="bg-gray-100">
      <th className="border p-2 text-left">Nome</th>
      <th className="border p-2 text-left">Matrícula</th>
    </tr>
  </thead>
  <tbody>
    {dados.map((item, i) => (
      <tr key={item.id} className={i % 2 === 0 ? 'bg-white' : 'bg-gray-50'}>
        <td className="border p-2">{item.nome}</td>
        <td className="border p-2">{item.matricula}</td>
      </tr>
    ))}
  </tbody>
</table>

// Rodapé
<div className="mt-6 text-xs text-gray-500 border-t pt-2">
  <p>ITP — Instituto Tiapretinha | Documento gerado automaticamente</p>
</div>
```

## Relatórios por Módulo

| Relatório | Dados | Role Mínima |
|-----------|-------|-------------|
| Lista de alunos | Nome, matrícula, turma, status | `assist` |
| Frequência por turma | Aluno, % presença, faltas | `prof` |
| Financeiro mensal | Receitas, despesas, saldo | `drt` |
| Estoque atual | Produto, quantidade, categoria | `assist` |
| Boletos em aberto | Aluno, valor, vencimento | `adjunto` |

## Busca de Alunos com Debounce

```tsx
import { useState, useEffect } from 'react';

const [search, setSearch] = useState('');
const [results, setResults] = useState([]);

useEffect(() => {
  const timer = setTimeout(async () => {
    if (search.length < 2) return;
    const { data } = await axios.get(`/backend-api/alunos?search=${search}`);
    setResults(data);
  }, 300);  // 300ms debounce

  return () => clearTimeout(timer);
}, [search]);
```

## Checklist de Relatório

- [ ] Cabeçalho com logo e data de geração
- [ ] Role verificada antes de exibir dados sensíveis
- [ ] Nome do arquivo inclui data (para versionamento)
- [ ] PDF em fundo branco (`bg-white text-black`) — não dark mode
- [ ] Excel com largura de colunas definida
- [ ] Loading state durante geração
- [ ] Tratamento de erro se geração falhar
