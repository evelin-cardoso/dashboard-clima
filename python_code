import streamlit as st
import pandas as pd
import requests
import plotly.express as px
import math

def local_css():
    st.markdown("""
    <style>
    /* Fundo e texto */
    .main {
        background-color: #f9f9f9;
        color: #222222;
        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    }
    h1 {
        color: #2c3e50;
        font-weight: 700;
        margin-bottom: 1.5rem;
        text-align: center;
        letter-spacing: 1.2px;
    }
    /* Espaçamento */
    .section {
        margin-top: 3rem;
        margin-bottom: 3rem;
        padding: 0 10px;
    }
    /* Cards com borda e separação visível */
    .metric-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
        gap: 18px;
        margin-top: 20px;
    }
    .metric-card {
        background: #ffffff;
        border: 1.5px solid #a3c4f3;  /* borda azul clara */
        border-radius: 12px;
        padding: 18px 20px;
        box-shadow: 0 3px 8px rgba(0,0,0,0.06);
        color: #34495e;
        text-align: center;
        font-weight: 600;
        transition: transform 0.2s ease, box-shadow 0.2s ease;
    }
    .metric-card:hover {
        transform: translateY(-3px);
        box-shadow: 0 7px 18px rgba(0,0,0,0.1);
    }
    .metric-card h4 {
        margin: 0 0 8px 0;
        font-size: 1rem;
        color: #2c3e50;
        font-weight: 700;
    }
    .metric-card p {
        margin: 0;
        font-size: 1.5rem;
        color: #2c3e50;
        font-weight: 700;
    }
    /* Cabeçalho da cidade */
    .city-header {
        display: flex;
        align-items: center;
        gap: 15px;
        margin-bottom: 12px;
        flex-wrap: wrap;
        justify-content: center;
    }
    .city-header img {
        border-radius: 15px;
        box-shadow: 0 4px 10px rgba(0,0,0,0.1);
    }
    .city-title {
        font-size: 1.6rem;
        font-weight: 800;
        color: #2c3e50;
        margin-bottom: 4px;
    }
    .city-subtitle {
        font-size: 0.9rem;
        color: #7f8c8d;
    }
    /* Análise texto dos gráficos */
    .analysis-text {
        font-style: italic;
        color: #555555;
        font-size: 0.95rem;
        margin-top: 5px;
        margin-bottom: 20px;
        max-width: 100%;
        text-align: center;
    }
    /* Layout geral */
    .block-container {
        padding-top: 2rem;
        padding-bottom: 2rem;
        max-width: 1200px;
        margin-left: auto;
        margin-right: auto;
    }
    /* Gráficos Plotly com fundo transparente */
    .js-plotly-plot .main-svg {
        background-color: transparent !important;
    }
    </style>
    """, unsafe_allow_html=True)

local_css()

st.set_page_config(page_title="Previsão do Tempo", layout="wide")

API_KEY = "88a7327d44db34f3782979267d132dda"

st.title("🌤️ Dashboard do Clima")
st.markdown("Visualização da previsão do tempo para várias cidades brasileiras.")

def pegar_dados(cidade):
    url = f"https://api.openweathermap.org/data/2.5/forecast?q={cidade}&appid={API_KEY}&units=metric&lang=pt_br"
    res = requests.get(url).json()
    if "list" not in res:
        st.error(f"Erro ao obter dados de {cidade}: {res.get('message', 'Erro desconhecido')}")
        st.stop()
    lista = res["list"]
    dados = []
    for item in lista:
        dados.append({
            "Data": item["dt_txt"],
            "Temperatura (°C)": item["main"]["temp"],
            "Sensação Térmica (°C)": item["main"]["feels_like"],
            "Umidade (%)": item["main"]["humidity"],
            "Descrição": item["weather"][0]["description"].capitalize(),
            "Ícone": item["weather"][0]["icon"],
            "Temp Mín (°C)": item["main"]["temp_min"],
            "Temp Máx (°C)": item["main"]["temp_max"],
            "Vento (m/s)": item["wind"]["speed"],
            "Direção do Vento (°)": item["wind"].get("deg", 0),
            "Pressão (hPa)": item["main"]["pressure"]
        })
    df = pd.DataFrame(dados)
    return df, res["city"]

def pegar_uv(lat, lon):
    url = f"http://api.openweathermap.org/data/2.5/uvi?appid={API_KEY}&lat={lat}&lon={lon}"
    res = requests.get(url).json()
    return res.get("value", None)

def calcular_ponto_orvalho(T, RH):
    a = 17.27
    b = 237.7
    alpha = ((a * T) / (b + T)) + math.log(RH/100.0)
    Td = (b * alpha) / (a - alpha)
    return Td

cidades = ["Florianópolis", "São Paulo", "Rio de Janeiro", "Curitiba", "Belo Horizonte"]

with st.container():
    col1, col2 = st.columns(2)
    with col1:
        cidade1 = st.selectbox("Selecione a primeira cidade:", cidades, key="cidade1")
    with col2:
        cidade2 = st.selectbox("Selecione a segunda cidade:", cidades, key="cidade2")

horas = st.slider("Quantas horas de previsão deseja ver?", min_value=12, max_value=48, step=12, value=24)

df1, info1 = pegar_dados(cidade1)
df2, info2 = pegar_dados(cidade2)

df1 = df1.head(horas // 3)
df2 = df2.head(horas // 3)

def formatar_data(df):
    df = df.copy()
    df["Data"] = pd.to_datetime(df["Data"]).dt.strftime("%d/%m %H:%M")
    return df

df1 = formatar_data(df1)
df2 = formatar_data(df2)

uv1 = pegar_uv(info1['coord']['lat'], info1['coord']['lon'])
uv2 = pegar_uv(info2['coord']['lat'], info2['coord']['lon'])

dew1 = calcular_ponto_orvalho(df1["Temperatura (°C)"].iloc[0], df1["Umidade (%)"].iloc[0])
dew2 = calcular_ponto_orvalho(df2["Temperatura (°C)"].iloc[0], df2["Umidade (%)"].iloc[0])

with st.container():
    st.markdown('<div class="section">', unsafe_allow_html=True)
    st.subheader("🌡️ Dados atuais das cidades selecionadas")
    col1, col2 = st.columns(2)

    def mostrar_dados_atualizados(info, df, uv, dew):
        st.markdown(f'''
            <div class="city-header">
                <img src="http://openweathermap.org/img/wn/{df['Ícone'][0]}@4x.png" width="90" alt="Ícone do clima">
                <div>
                    <div class="city-title">📍 {info['name']} - {info['country']}</div>
                    <div class="city-subtitle">🌍 Coordenadas: {info['coord']['lat']:.3f}, {info['coord']['lon']:.3f}</div>
                </div>
            </div>
            <p style="text-align:center; font-size:1.1rem; color:#34495e; margin-top:-10px; font-weight:600;">
                {df['Descrição'][0]}
            </p>
        ''', unsafe_allow_html=True)

        st.markdown('<div class="metric-grid">', unsafe_allow_html=True)

        metrics = [
            ("Temperatura (°C)", f"{df['Temperatura (°C)'][0]:.1f}"),
            ("Sensação Térmica (°C)", f"{df['Sensação Térmica (°C)'][0]:.1f}"),
            ("Umidade (%)", f"{df['Umidade (%)'][0]:.0f}"),
            ("Ponto de Orvalho (°C)", f"{dew:.1f}"),
            ("Índice UV", f"{uv if uv is not None else 'N/A'}"),
            ("Pressão (hPa)", f"{df['Pressão (hPa)'][0]}"),
            ("Vento (m/s)", f"{df['Vento (m/s)'][0]:.1f}"),
            ("Direção do Vento (°)", f"{df['Direção do Vento (°)'][0]}"),
        ]

        for nome, valor in metrics:
            st.markdown(f'''
                <div class="metric-card">
                    <h4>{nome}</h4>
                    <p>{valor}</p>
                </div>
            ''', unsafe_allow_html=True)

        st.markdown('</div>', unsafe_allow_html=True)

    with col1:
        mostrar_dados_atualizados(info1, df1, uv1, dew1)
    with col2:
        mostrar_dados_atualizados(info2, df2, uv2, dew2)

    st.markdown('</div>', unsafe_allow_html=True)

# Análises para os gráficos
def analise_temperatura(df, cidade):
    temp_max = df["Temp Máx (°C)"].max()
    temp_min = df["Temp Mín (°C)"].min()
    if temp_max - temp_min > 10:
        return f"A previsão para {cidade} indica variação significativa de temperatura entre {temp_min:.1f}°C e {temp_max:.1f}°C."
    else:
        return f"A temperatura em {cidade} deve se manter relativamente estável, com mínima de {temp_min:.1f}°C e máxima de {temp_max:.1f}°C."

def analise_sensacao(df, cidade):
    sensacao_max = df["Sensação Térmica (°C)"].max()
    sensacao_min = df["Sensação Térmica (°C)"].min()
    if sensacao_max > 30:
        return f"Prepare-se para uma sensação térmica elevada em {cidade}, chegando a {sensacao_max:.1f}°C."
    elif sensacao_min < 10:
        return f"A sensação térmica pode ficar fria em {cidade}, chegando a {sensacao_min:.1f}°C."
    else:
        return f"A sensação térmica em {cidade} permanece agradável."

def analise_umidade(df, cidade):
    umidade_min = df["Umidade (%)"].min()
    if umidade_min < 30:
        return f"A umidade em {cidade} pode ficar baixa, com níveis mínimos de {umidade_min:.0f}%, fique atento à hidratação."
    else:
        return f"A umidade em {cidade} estará em níveis confortáveis."

# Gráficos com fundos transparentes e eixos escuros
cores_temp = ['#2980b9', '#3498db', '#85c1e9']
cor_sensacao = ['#d35400']
cor_umidade = ['#3498db']

st.markdown('<div class="section">', unsafe_allow_html=True)
st.subheader("📊 Comparação das temperaturas")

col1, col2 = st.columns(2)
with col1:
    fig1 = px.line(df1, x="Data", y=["Temperatura (°C)", "Temp Máx (°C)", "Temp Mín (°C)"],
                   title=f"Temperaturas em {cidade1}",
                   labels={"value":"Temperatura (°C)", "Data":"Data"},
                   color_discrete_sequence=cores_temp)
    fig1.update_layout(
        template="plotly_white",
        xaxis_tickangle=-45,
        plot_bgcolor='rgba(0,0,0,0)',
        paper_bgcolor='rgba(0,0,0,0)',
        xaxis=dict(color='#2c3e50'),
        yaxis=dict(color='#2c3e50')
    )
    st.plotly_chart(fig1, use_container_width=True)
    st.markdown(f'<p class="analysis-text">{analise_temperatura(df1, cidade1)}</p>', unsafe_allow_html=True)
with col2:
    fig2 = px.line(df2, x="Data", y=["Temperatura (°C)", "Temp Máx (°C)", "Temp Mín (°C)"],
                   title=f"Temperaturas em {cidade2}",
                   labels={"value":"Temperatura (°C)", "Data":"Data"},
                   color_discrete_sequence=cores_temp)
    fig2.update_layout(
        template="plotly_white",
        xaxis_tickangle=-45,
        plot_bgcolor='rgba(0,0,0,0)',
        paper_bgcolor='rgba(0,0,0,0)',
        xaxis=dict(color='#2c3e50'),
        yaxis=dict(color='#2c3e50')
    )
    st.plotly_chart(fig2, use_container_width=True)
    st.markdown(f'<p class="analysis-text">{analise_temperatura(df2, cidade2)}</p>', unsafe_allow_html=True)
st.markdown('</div>', unsafe_allow_html=True)

st.markdown('<div class="section">', unsafe_allow_html=True)
st.subheader("🔥 Comparação da sensação térmica")

col1, col2 = st.columns(2)
with col1:
    fig3 = px.line(df1, x="Data", y="Sensação Térmica (°C)", color_discrete_sequence=cor_sensacao,
                   title=f"Sensação térmica em {cidade1}",
                   labels={"Sensação Térmica (°C)":"Sensação Térmica (°C)", "Data":"Data"})
    fig3.update_layout(
        template="plotly_white",
        xaxis_tickangle=-45,
        plot_bgcolor='rgba(0,0,0,0)',
        paper_bgcolor='rgba(0,0,0,0)',
        xaxis=dict(color='#2c3e50'),
        yaxis=dict(color='#2c3e50')
    )
    st.plotly_chart(fig3, use_container_width=True)
    st.markdown(f'<p class="analysis-text">{analise_sensacao(df1, cidade1)}</p>', unsafe_allow_html=True)
with col2:
    fig4 = px.line(df2, x="Data", y="Sensação Térmica (°C)", color_discrete_sequence=cor_sensacao,
                   title=f"Sensação térmica em {cidade2}",
                   labels={"Sensação Térmica (°C)":"Sensação Térmica (°C)", "Data":"Data"})
    fig4.update_layout(
        template="plotly_white",
        xaxis_tickangle=-45,
        plot_bgcolor='rgba(0,0,0,0)',
        paper_bgcolor='rgba(0,0,0,0)',
        xaxis=dict(color='#2c3e50'),
        yaxis=dict(color='#2c3e50')
    )
    st.plotly_chart(fig4, use_container_width=True)
    st.markdown(f'<p class="analysis-text">{analise_sensacao(df2, cidade2)}</p>', unsafe_allow_html=True)
st.markdown('</div>', unsafe_allow_html=True)

st.markdown('<div class="section">', unsafe_allow_html=True)
st.subheader("💧 Comparação da umidade")

col1, col2 = st.columns(2)
with col1:
    fig5 = px.bar(df1, x="Data", y="Umidade (%)", title=f"Umidade em {cidade1}",
                  labels={"Umidade (%)": "Umidade (%)", "Data":"Data"},
                  color_discrete_sequence=cor_umidade)
    fig5.update_layout(
        template="plotly_white",
        xaxis_tickangle=-45,
        plot_bgcolor='rgba(0,0,0,0)',
        paper_bgcolor='rgba(0,0,0,0)',
        xaxis=dict(color='#2c3e50'),
        yaxis=dict(color='#2c3e50')
    )
    st.plotly_chart(fig5, use_container_width=True)
    st.markdown(f'<p class="analysis-text">{analise_umidade(df1, cidade1)}</p>', unsafe_allow_html=True)
with col2:
    fig6 = px.bar(df2, x="Data", y="Umidade (%)", title=f"Umidade em {cidade2}",
                  labels={"Umidade (%)": "Umidade (%)", "Data":"Data"},
                  color_discrete_sequence=cor_umidade)
    fig6.update_layout(
        template="plotly_white",
        xaxis_tickangle=-45,
        plot_bgcolor='rgba(0,0,0,0)',
        paper_bgcolor='rgba(0,0,0,0)',
        xaxis=dict(color='#2c3e50'),
        yaxis=dict(color='#2c3e50')
    )
    st.plotly_chart(fig6, use_container_width=True)
    st.markdown(f'<p class="analysis-text">{analise_umidade(df2, cidade2)}</p>', unsafe_allow_html=True)
st.markdown('</div>', unsafe_allow_html=True)

# Mapa das cidades selecionadas

st.markdown('<div class="section">', unsafe_allow_html=True)
st.subheader("🌍 Mapa das Cidades Selecionadas")

mapa_df = pd.DataFrame({
    "Cidade": [cidade1, cidade2],
    "Latitude": [info1['coord']['lat'], info2['coord']['lat']],
    "Longitude": [info1['coord']['lon'], info2['coord']['lon']],
    "Temperatura (°C)": [df1["Temperatura (°C)"].iloc[0], df2["Temperatura (°C)"].iloc[0]],
    "Descrição": [df1["Descrição"].iloc[0], df2["Descrição"].iloc[0]]
})

fig_map = px.scatter_mapbox(mapa_df, lat="Latitude", lon="Longitude", hover_name="Cidade",
                            hover_data={"Temperatura (°C)": True, "Descrição": True},
                            color="Temperatura (°C)", size="Temperatura (°C)",
                            color_continuous_scale=px.colors.sequential.Tealgrn, size_max=25, zoom=4)

fig_map.update_layout(
    mapbox_style="open-street-map",
    margin={"r":0,"t":0,"l":0,"b":0},
    template="plotly_white",
    paper_bgcolor='rgba(0,0,0,0)',
    plot_bgcolor='rgba(0,0,0,0)'
)

st.plotly_chart(fig_map, use_container_width=True)
st.markdown('</div>', unsafe_allow_html=True)

