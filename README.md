# Trabalho-2-Trim
#Trabalho para o professor Júlio Pires.Entrada e saída de dados
#Alunos: Júlia de Souza Águiar, Kauã César Alves de Sousa .
import tkinter as tk
from tkinter import messagebox, scrolledtext, filedialog, simpledialog
import requests
from bs4 import BeautifulSoup
from textblob import TextBlob
from textblob import Word as TextBlobWord
import spacy
import nltk
from nltk.corpus import stopwords
from nltk.stem import PorterStemmer

try:
    nltk.data.find('corpora/stopwords')
except LookupError:
    nltk.download('stopwords')
    nltk.download('wordnet')

try:
    analisador_spacy = spacy.load("en_core_web_sm")
except OSError:
    import os
    os.system("python -m spacy download en_core_web_sm")
    analisador_spacy = spacy.load("en_core_web_sm")


def buscar_arquivo_local():
    caminho_arquivo = filedialog.askopenfilename(filetypes=[("Arquivos de Texto", "*.txt")])
    if caminho_arquivo:
        with open(caminho_arquivo, 'r', encoding='utf-8') as arquivo:
            campo_texto.delete('1.0', tk.END)
            campo_texto.insert(tk.END, arquivo.read())

def importar_texto_da_web():
    link_web = simpledialog.askstring("Importar da Web", "Cole o link da página aqui:")
    if link_web:
        try:
            requisicao = requests.get(link_web, timeout=10)
            pagina = BeautifulSoup(requisicao.text, 'html.parser')
            for elemento_inutil in pagina(["script", "style"]):
                elemento_inutil.decompose()
            
            linhas_texto = (linha.strip() for linha in pagina.get_text().splitlines())
            blocos_texto = (trecho.strip() for linha in linhas_texto for trecho in linha.split(" "))
            texto_final = '\n'.join(bloco for bloco in blocos_texto if bloco)
            
            campo_texto.delete('1.0', tk.END)
            campo_texto.insert(tk.END, texto_final)
        except Exception as erro:
            messagebox.showerror("Houve um problema", f"Não conseguimos acessar esse link:\n{erro}")

def pegar_texto_atual():
    conteudo = campo_texto.get('1.0', tk.END).strip()
    if not conteudo:
        messagebox.showwarning("Texto ausente", "Por favor, digite algo ou carregue um arquivo primeiro.")
        return None
    return conteudo

def mostrar_resultado(aba, dados):
    campo_resultados.delete('1.0', tk.END)
    campo_resultados.insert(tk.END, f"=== {aba} ===\n\n{dados}")


def analisar_tokens():
    texto = pegar_texto_atual()
    if not texto: return
    documento = TextBlob(texto)
    
    frases = documento.sentences
    palavras = documento.words
    
    resultado = f"Contagem Geral:\n• Total de Frases: {len(frases)}\n• Total de Palavras: {len(palavras)}\n\n"
    resultado += "--- Lista de Frases Encontradas ---\n" + "\n".join([f"• {f}" for f in frases])
    mostrar_resultado("Divisão do Texto (Tokenização)", resultado)

def mapear_classes_gramaticais():
    texto = pegar_texto_atual()
    if not texto: return
    documento = TextBlob(texto)
    resultado = "\n".join([f"Palavra: '{palavra}' -> Categoria: {classe}" for palavra, classe in documento.tags])
    mostrar_resultado("Análise Gramatical", resultado)

def extrair_sintagmas():
    texto = pegar_texto_atual()
    if not texto: return
    documento = TextBlob(texto)
    resultado = "\n".join([f"• {bloco}" for bloco in documento.noun_phrases])
    if not resultado: resultado = "Não encontramos blocos nominais específicos."
    mostrar_resultado("Sintagmas Nominais", resultado)


def avaliar_sentimento():
    texto = pegar_texto_atual()
    if not texto: return
    documento = TextBlob(texto)
    tom = documento.sentiment.polarity
    subjetividade = documento.sentiment.subjectivity
    
    if tom > 0.1: clima_texto = "Predominantemente Positivo"
    elif tom < -0.1: clima_texto = "Predominantemente Negativo"
    else: clima_texto = "Predominantemente Neutro"
    
    resultado = f"Métricas de Sentimento:\n"
    resultado += f"• Humor/Tom: {tom:.2f} (Varia de -1 a 1)\n"
    resultado += f"• Subjetividade: {subjetividade:.2f} (Varia de 0 a 1)\n\n"
    resultado += f"Diagnóstico: Este texto parece ser {clima_texto}."
    mostrar_resultado("Análise de Sentimento", resultado)


def alternar_plural_singular():
    texto = pegar_texto_atual()
    if not texto: return
    documento = TextBlob(texto)
    resultado = "Variações de número das palavras (Amostra):\n\n"
    for item in list(set(documento.words))[:20]:
        palavra_objeto = TextBlobWord(item)
        resultado += f"Original: {item} | Plural: {palavra_objeto.pluralize()} | Singular: {palavra_objeto.singularize()}\n"
    mostrar_resultado("Flexão de Palavras", resultado)

def sugerir_correcao():
    texto = pegar_texto_atual()
    if not texto: return
    documento = TextBlob(texto)
    texto_ajustado = documento.correct()
    resultado = f"--- Sugestão de Texto Corrigido ---\n\n{texto_ajustado}"
    mostrar_resultado("Correção de Ortografia", resultado)

def comparar_normalizacao():
    texto = pegar_texto_atual()
    if not texto: return
    documento = TextBlob(texto)
    redutor_stem = PorterStemmer()
    
    resultado = f"{'Palavra':<20} | {'Forma Radical':<20} | {'Forma Dicionário':<20}\n"
    resultado += "-" * 70 + "\n"
    for item in list(set(documento.words))[:20]:
        palavra_objeto = TextBlobWord(item)
        resultado += f"{item:<20} | {redutor_stem.stem(item):<20} | {palavra_objeto.lemmatize('v'):<20}\n"
    mostrar_resultado("Normalização (Stemming vs Lemmatization)", resultado)


def contar_frequencia_palavras():
    texto = pegar_texto_atual()
    if not texto: return
    documento = TextBlob(texto)
    contagem = documento.word_counts
    ranking = sorted(contagem.items(), key=lambda x: x[1], reverse=True)
    
    resultado = "Palavras mais utilizadas no texto:\n\n"
    for palavra, qtd in ranking[:10]:
        resultado += f"• {palavra}: usada {qtd} vezes\n"
    mostrar_resultado("Top 10 Palavras Recorrentes", resultado)

def pesquisar_no_wordnet():
    termo = simpledialog.askstring("Dicionário WordNet", "Digite uma palavra em inglês:")
    if not termo: return
    palavra_objeto = TextBlobWord(termo)
    
    resultado = f"Análise da Palavra: {termo}\n\n"
    resultado += "--- Definições Principais ---\n"
    for definicao in palavra_objeto.synsets[:3]:
        resultado += f"• ({definicao.pos()}): {definicao.definition()}\n"
        
    sinonimos = set()
    antonimos = set()
    for definicao in palavra_objeto.synsets:
        for lema in definicao.lemmas():
            sinonimos.add(lema.name())
            if lema.antonyms():
                antonimos.add(lema.antonyms()[0].name())
                
    resultado += f"\nSinônimos Próximos: {', '.join(list(sinonimos)[:10]) if sinonimos else 'Nenhum encontrado'}\n"
    resultado += f"Antônimos Opostos: {', '.join(list(antonimos)[:10]) if antonimos else 'Nenhum encontrado'}\n"
    mostrar_resultado("Pesquisa Dicionário WordNet", resultado)

def limpar_stop_words():
    texto = pegar_texto_atual()
    if not texto: return
    palavras_vazias = set(stopwords.words('english'))
    documento = TextBlob(texto)
    
    filtrado = [p for p in documento.words if p.lower() not in palavras_vazias]
    resultado = "Texto limpo (sem artigos, preposições ou conectivos):\n\n" + " ".join(filtrado)
    mostrar_resultado("Remoção de Conectivos (Stop Words)", resultado)

def construir_ngrams():
    texto = pegar_texto_atual()
    if not texto: return
    tamanho_n = simpledialog.askinteger("Configurar n-gramas", "Agrupar de quantas em quantas palavras?", minvalue=1, maxvalue=5)
    if not tamanho_n: return
    
    documento = TextBlob(texto)
    grupos = documento.ngrams(tamanho_n)
    
    resultado = f"--- Sequências de {tamanho_n} palavras geradas ---\n\n"
    for g in grupos:
        resultado += f"• {list(g)}\n"
    mostrar_resultado(f"Geração de {tamanho_n}-gramas", resultado)


def mapear_entidades():
    texto = pegar_texto_atual()
    if not texto: return
    processamento = analisador_spacy(texto)
    
    resultado = ""
    for entidade in processamento.ents:
        resultado += f"Termo: {entidade.text:<25} | Tipo: {entidade.label_:<15} | Significado: {spacy.explain(entidade.label_)}\n"
        
    if not resultado: resultado = "Não encontramos nenhuma entidade específica (nomes, locais, datas)."
    mostrar_resultado("Entidades Nomeadas Identificadas (NER)", resultado)

def medir_similaridade():
    texto_original = pegar_texto_atual()
    if not texto_original: return
    segundo_texto = simpledialog.askstring("Comparar Textos", "Cole o segundo texto para comparar:")
    if not segundo_texto: return
    
    doc1 = analisador_spacy(texto_original)
    doc2 = analisador_spacy(segundo_texto)
    pontuacao = doc1.similarity(doc2)
    
    resultado = f"Texto Base: {texto_original[:60]}...\n"
    resultado += f"Texto Comparado: {segundo_texto[:60]}...\n\n"
    resultado += f"Proximidade de Significado Semântico: {pontuacao:.4f} (Escala de 0.0 a 1.0)"
    mostrar_resultado("Cálculo de Similaridade Semântica", resultado)


janela = tk.Tk()
janela.title("Painel de Inteligência Artificial Aplicada - NLP")
janela.geometry("950x720")

painel_entrada = tk.LabelFrame(janela, text=" 1. Onde está o seu texto? ")
painel_entrada.pack(fill=tk.X, padx=10, pady=5)

tk.Button(painel_entrada, text="Escolher arquivo de texto (.txt)", command=buscar_arquivo_local).pack(side=tk.LEFT, padx=5, pady=5)
tk.Button(painel_entrada, text="Puxar texto de uma URL/Site", command=importar_texto_da_web).pack(side=tk.LEFT, padx=5, pady=5)

campo_texto = scrolledtext.ScrolledText(janela, height=8, wrap=tk.WORD)
campo_texto.pack(fill=tk.X, padx=10, pady=5)
campo_texto.insert(tk.END, "Type or paste your English text here for analysis.")

painel_ferramentas = tk.LabelFrame(janela, text=" 2. Escolha uma Análise de IA ")
painel_ferramentas.pack(fill=tk.X, padx=10, pady=5)

linha_a = tk.Frame(painel_ferramentas)
linha_a.pack(anchor=tk.W, pady=2)
tk.Label(linha_a, text="Estrutura Básica:", width=18, anchor=tk.W).pack(side=tk.LEFT, padx=5)
tk.Button(linha_a, text="Contar Palavras/Frases", command=analisar_tokens).pack(side=tk.LEFT, padx=2)
tk.Button(linha_a, text="Categorias Gramaticais", command=mapear_classes_gramaticais).pack(side=tk.LEFT, padx=2)
tk.Button(linha_a, text="Sintagmas Nominais", command=extrair_sintagmas).pack(side=tk.LEFT, padx=2)

linha_b = tk.Frame(painel_ferramentas)
linha_b.pack(anchor=tk.W, pady=2)
tk.Label(linha_b, text="Sentimento:", width=18, anchor=tk.W).pack(side=tk.LEFT, padx=5)
tk.Button(linha_b, text="Analisar Humor do Texto", command=avaliar_sentimento).pack(side=tk.LEFT, padx=2)

linha_c = tk.Frame(painel_ferramentas)
linha_c.pack(anchor=tk.W, pady=2)
tk.Label(linha_c, text="Modificações:", width=18, anchor=tk.W).pack(side=tk.LEFT, padx=5)
tk.Button(linha_c, text="Testar Plural/Singular", command=alternar_plural_singular).pack(side=tk.LEFT, padx=2)
tk.Button(linha_c, text="Corrigir Erros de Escrita", command=sugerir_correcao).pack(side=tk.LEFT, padx=2)
tk.Button(linha_c, text="Reduzir Palavras às Raízes", command=comparar_normalizacao).pack(side=tk.LEFT, padx=2)

linha_d = tk.Frame(painel_ferramentas)
linha_d.pack(anchor=tk.W, pady=2)
tk.Label(linha_d, text="Estatísticas:", width=18, anchor=tk.W).pack(side=tk.LEFT, padx=5)
tk.Button(linha_d, text="Palavras Mais Usadas", command=contar_frequencia_palavras).pack(side=tk.LEFT, padx=2)
tk.Button(linha_d, text="Buscar no Dicionário Inteligente", command=pesquisar_no_wordnet).pack(side=tk.LEFT, padx=2)
tk.Button(linha_d, text="Filtrar Palavras Conectivas", command=limpar_stop_words).pack(side=tk.LEFT, padx=2)
tk.Button(linha_d, text="Agrupar Sequências (n-gramas)", command=construir_ngrams).pack(side=tk.LEFT, padx=2)

linha_e = tk.Frame(painel_ferramentas)
linha_e.pack(anchor=tk.W, pady=2)
tk.Label(linha_e, text="Modelos Avançados:", width=18, anchor=tk.W).pack(side=tk.LEFT, padx=5)
tk.Button(linha_e, text="Mapear Nomes/Locais/Datas (NER)", command=mapear_entidades).pack(side=tk.LEFT, padx=2)
tk.Button(linha_e, text="Testar Proximidade entre 2 Textos", command=medir_similaridade).pack(side=tk.LEFT, padx=2)

painel_resultados = tk.LabelFrame(janela, text=" 3. Diagnóstico e Resultados da IA ")
painel_resultados.pack(fill=tk.BOTH, expand=True, padx=10, pady=5)

campo_resultados = scrolledtext.ScrolledText(painel_resultados, wrap=tk.WORD, font=("Courier New", 10))
campo_resultados.pack(fill=tk.BOTH, expand=True, padx=5, pady=5)

janela.mainloop()
