# Modelação e Análise Espacial da Suscetibilidade a Incêndios Florestais em Portugal Continental (2024)

**Autor:** Áureo Miguel Rodrigues de Almeida
**Curso:** MSc Geographic Information Science and Systems, NOVA IMS · U.C. Base de Dados Geospaciais (2025)

## Resumo

Desenvolvimento de uma base de dados espacial relacional e de um modelo multicritério (MCDA) para avaliar a suscetibilidade a incêndios florestais em Portugal Continental, com foco no ano de 2024. A metodologia integra dados de áreas ardidas, uso do solo, topografia e clima numa base de dados PostgreSQL/PostGIS, seguida de análise espacial em QGIS e produção de uma carta de suscetibilidade.

## Dados utilizados

| Dataset | Fonte | Formato |
|---|---|---|
| Áreas ardidas 2024 | Copernicus/EFFIS | Shapefile |
| Uso do solo (COS 2018) | DGT | Shapefile, 1:25.000 |
| Modelo Digital de Terreno | Copernicus | GeoTIFF, 25 m |
| Temperatura/precipitação verão 2024 | IPMA/Copernicus CDS | Raster/CSV, 1 km |
| Rede viária e hidrografia | DGT | Shapefile, 1:25.000 |
| Limites administrativos (CAOP 2024) | DGT | Shapefile |

Sistema de referência: **EPSG:3763 (ETRS89/PT-TM06)**.

## Modelo de dados

Base de dados relacional com **7 tabelas (5 espaciais)** implementada em PostgreSQL 15 + PostGIS 3.4:

- `distrito`, `municipio` — hierarquia administrativa (NUTS III)
- `incendio` — ocorrências pontuais (geometry Point)
- `area_ardida` — áreas queimadas (geometry MultiPolygon)
- `uso_solo` — classificação COS 2018
- `classe_declive` — domínio de reclassificação topográfica
- `fatores_condicionantes` — variáveis ambientais/antrópicas por município

5 índices espaciais GIST, chaves primárias/estrangeiras e validação de geometrias.

```sql
CREATE TABLE area_ardida (
    id_area SERIAL PRIMARY KEY,
    id_incendio INTEGER REFERENCES incendio(id_incendio),
    id_uso_solo INTEGER REFERENCES uso_solo(id_uso_solo),
    area_ha DECIMAL(10,2) NOT NULL CHECK (area_ha > 0),
    data_ardida DATE,
    geom GEOMETRY(MultiPolygon, 3763)
);
CREATE INDEX idx_area_ardida_geom ON area_ardida USING GIST(geom);
```

## Metodologia (MCDA — Weighted Linear Combination)

Após derivação de variáveis no QGIS (declive via *Slope*, distância à rede viária via *Proximity/GDAL*, estatísticas zonais para clima), os fatores foram normalizados (0–10) e combinados por álgebra de mapas:

```
Suscetibilidade = 0.30·Uso_Solo + 0.25·Declive + 0.20·Temperatura + 0.15·Dist_Vias + 0.10·Precipitação
```

Classificação final em 5 classes por Natural Breaks (Jenks).

## Resultados

- Área ardida em 2024 (Copernicus/EFFIS): **140.362 ha**, 676 ocorrências
- As classes de suscetibilidade **alta/muito alta** concentram-se no interior norte e centro
- Os distritos de **Aveiro, Porto, Viseu e Braga concentraram 63% da área ardida em 2024**, coincidindo com as zonas identificadas como mais suscetíveis pelo modelo
- Modelos MCDA como este apresentam tipicamente taxas de acerto superiores a 65% quando validados com dados reais (literatura consultada)

## Stack

PostgreSQL 15 · PostGIS 3.4 · QGIS 3.34 · pgAdmin · DB Manager · Raster Calculator · Zonal Statistics · shp2pgsql

## Nota

O trabalho foi desenvolvido através de operações SQL diretas e da interface gráfica do QGIS (DB Manager, Raster Calculator). O relatório completo, com o diagrama Entidade-Atributo-Relação e todas as figuras, está incluído neste repositório (`relatorio.pdf`).
