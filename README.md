import streamlit as st

# Configuração da página do aplicativo
st.set_page_config(page_title="Gestor Operacional Scania 560R", layout="wide")

st.title("🚛 Gestor Operacional - Scania 560R 6x4 Super")
st.markdown("Altere os valores abaixo para recalcular custos e metas operacionais em tempo real.")

# Criando duas colunas na tela para organizar o aplicativo
col_dados, col_resultados = st.columns([1, 1])

with col_dados:
    st.header("📥 Dados de Entrada")
    
    st.subheader("Veículo e Capital")
    preco_caminhao = st.number_input("Preço do Caminhão Completo (R$)", value=1135000)
    depreciacao_pct = st.slider("Percentual de Depreciação em 5 anos (%)", min_value=10, max_value=60, value=35) / 100
    prazo_meses = st.number_input("Prazo para Amortização (Meses)", value=60)
    
    st.subheader("Custos de Viagem")
    preco_diesel = st.number_input("Preço do Litro do Diesel S10 (R$)", value=7.28, step=0.1)
    consumo_kml = st.number_input("Consumo Médio Carregado (KM/L)", value=2.45, step=0.05)
    preco_arla = st.number_input("Preço do Litro do Arla 32 (R$)", value=3.50, step=0.1)
    valor_frete = st.number_input("Valor Médio Líquido do Frete (R$/KM)", value=7.50, step=0.5)
    
    st.subheader("Custos Fixos da Operação")
    motorista = st.number_input("Motorista (Salário + Encargos) (R$/mês)", value=11000)
    seguro = st.number_input("Seguro Total + Rastreador (R$/mês)", value=3600)
    manutencao_km = st.number_input("Manutenção e Pneus por KM (R$)", value=0.55, step=0.05)

# Processamento de Cálculos Internos do Aplicativo
valor_depreciacao = preco_caminhao * depreciacao_pct
valor_revenda = preco_caminhao - valor_depreciacao
ipva_mensal = (preco_caminhao * 0.015) / 12
depreciacao_mensal = valor_depreciacao / prazo_meses

total_custos_fixos = motorista + seguro + ipva_mensal + depreciacao_mensal

custo_diesel_km = preco_diesel / consumo_kml
custo_arla_km = custo_diesel_km * 0.06 * preco_arla
total_custos_variaveis_km = custo_diesel_km + custo_arla_km + manutencao_km

sobra_liquida_km = valor_frete - total_custos_variaveis_km

# Prevenção contra divisão por zero
if sobra_liquida_km > 0:
    km_empatar = total_custos_fixos / sobra_liquida_km
    km_lucro = (total_custos_fixos + (preco_caminhao / prazo_meses)) / sobra_liquida_km
    km_diario = km_lucro / 30
else:
    km_empatar = km_lucro = km_diario = 0

with col_resultados:
    st.header("📊 Resultados Operacionais")
    
    # Exibição dos indicadores em formato de cartões de aplicativo
    st.metric(label="Sobra Líquida por Quilômetro", value=f"R$ {sobra_liquida_km:.2f} por KM")
    
    col_f1, col_f2 = st.columns(2)
    with col_f1:
        st.metric(label="Total de Custos Fixos Mensais", value=f"R$ {total_custos_fixos:.2f}")
    with col_f2:
        st.metric(label="Custos Variáveis por KM", value=f"R$ {total_custos_variaveis_km:.2f}")
        
    st.subheader("🎯 Metas de Quilometragem para o Motorista")
    st.error(f"**KM Mensal Mínimo para Empatar:** {int(km_empatar):,} KM/mês")
    st.success(f"**KM Mensal para Se Pagar + Dar Lucro:** {int(km_lucro):,} KM/mês")
    st.info(f"**Meta Recomendada de Rodagem Diária:** {int(km_diario)} KM/dia")
    
    st.subheader("📉 Previsão do Patrimônio (5 anos)")
    st.write(f"• **Perda de Valor do Caminhão (Depreciação):** R$ {valor_depreciacao:,.2f}")
    st.write(f"• **Valor Estimado de Venda Futura:** R$ {valor_revenda:,.2f}")
    
    if sobra_liquida_km <= 0:
        st.warning("⚠️ Atenção: O custo por KM está maior que o valor do frete! A operação está operando no prejuízo.")
