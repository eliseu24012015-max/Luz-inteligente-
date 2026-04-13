import streamlit as st
import PyPDF2
import re

st.set_page_config(page_title="Analisador de Conta de Luz")

st.title("⚡ Análise de Conta de Energia")

arquivo = st.file_uploader("Envie sua conta em PDF", type="pdf")


def extrair_texto(pdf):
    leitor = PyPDF2.PdfReader(pdf)
    texto = ""

    for pagina in leitor.pages:
        texto += pagina.extract_text()

    return texto


if arquivo:

    texto = extrair_texto(arquivo)

    st.subheader("Texto Extraído:")
    st.text_area("", texto, height=300)

    numeros = re.findall(r'\d+[.,]?\d*', texto)

    numeros_limpos = []

    for num in numeros:
        try:
            numeros_limpos.append(float(num.replace(",", ".")))
        except:
            pass

    consumos = [n for n in numeros_limpos if 50 <= n <= 1000]

    if consumos:

        st.success("Consumos encontrados!")

        st.subheader("📊 Histórico de Consumo:")

        for i, consumo in enumerate(consumos[:12]):
            st.write(f"Mês {i+1}: {consumo} kWh")

        media = sum(consumos[:12]) / len(consumos[:12])

        st.subheader("📈 Média de Consumo")

        st.metric("Média Mensal", f"{media:.2f} kWh")

        st.subheader("💰 Estudo de Economia")

        economia_10 = media * 0.10
        economia_20 = media * 0.20

        st.write(f"Reduzindo 10% do consumo: **{economia_10:.2f} kWh/mês**")
        st.write(f"Reduzindo 20% do consumo: **{economia_20:.2f} kWh/mês**")

        if media > 300:
            st.warning("⚠ Consumo elevado detectado.")
            st.write("Sugestão: avaliar troca para iluminação LED e revisar ar-condicionado.")

        else:
            st.success("✅ Seu consumo está dentro de uma faixa razoável.")

    else:
        st.error("Nenhum consumo identificado.")
