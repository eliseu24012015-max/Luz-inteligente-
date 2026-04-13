import streamlit as st
import PyPDF2
import re
from io import BytesIO

# CONFIGURAÇÃO DA PÁGINA
st.set_page_config(
    page_title="Leitor de Conta de Luz",
    page_icon="⚡",
    layout="centered"
)

st.title("⚡ Leitor de Conta de Energia PDF")
st.write("Faça upload de uma conta de luz em PDF para localizar o consumo em kWh.")

# FUNÇÃO PARA EXTRAIR TEXTO DO PDF
def extrair_texto_pdf(pdf_file):
    texto = ""

    try:
        leitor = PyPDF2.PdfReader(pdf_file)

        for pagina in leitor.pages:
            texto += pagina.extract_text() + "\n"

        return texto

    except Exception as e:
        st.error(f"Erro ao ler PDF: {e}")
        return None


# UPLOAD
arquivo = st.file_uploader(
    "Selecione o PDF da conta de energia",
    type=["pdf"]
)

if arquivo is not None:

    st.success("PDF enviado com sucesso!")

    texto_pdf = extrair_texto_pdf(arquivo)

    if texto_pdf:

        st.subheader("Texto Extraído:")
        st.text_area("", texto_pdf, height=300)

        # BUSCAR KWH
        if "kWh" in texto_pdf or "KWH" in texto_pdf or "kwh" in texto_pdf:

            st.success("✅ Palavra 'kWh' encontrada no documento!")

            # Regex para encontrar valores próximos de kWh
            padrao = r'(\d+[.,]?\d*)\s*kWh'
            resultados = re.findall(padrao, texto_pdf, re.IGNORECASE)

            if resultados:
                st.subheader("🔍 Possíveis consumos encontrados:")

                for i, valor in enumerate(resultados):
                    st.write(f"Consumo {i+1}: **{valor} kWh**")

            else:
                st.warning("Palavra encontrada, mas nenhum valor associado detectado.")

        else:
            st.error("❌ Palavra 'kWh' não encontrada no PDF.")
