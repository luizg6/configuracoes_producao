# EDGV 3.0 Topo 1.3: Models QGIS Elemento Intersecção

------------------------------------

Modelos construídos para a produção EDGV 3.0 Topo versão 1.3, na linha de produção de Conjunto de Dados Geoespaciais Vetoriais.

## Classes utilizadas

- infra_elemento_viario_p;
- infra_ferrovia_l;
- infra_via_deslocamento_l;
- infra_mobilidade_urbana_l;
- elemnat_trecho_drenagem_l;
- infra_elemento_viario_l;
- infra_travessia_hidroviaria_l;
- infra_barragem_l
- elemnat_trecho_drenagem_l.
- aux_moldura_a;
- moldura;
- aux_moldura_area_continua_a;


### Expressão para capturar todas as geometrias carregadas

```
array_to_string ( array_foreach ( array_filter ( array_filter (@layers,not (regexp_match (layer_property (@element,'name'), '(rascunho|rev_|val_|aux_|moldura|Flags|flags)'))), layer_property (@element,'geometry_type') in ('Polygon','Line', 'Point')), layer_property (@element,'name')))
```

## Ordem dos processos

1. Remover geometrias nulas / Desagregar geometrias / Remover vértices duplicados / Remover feições duplicadas / identify features with invalid unicode;
2. Identificar Geometrias inválidas (com correção automática) / Identificar ângulos pequenos;
3. Unir linhas de mesmo conjunto de atributos;
4. Identificar linhas entrelaçadas;
5. Limpeza topológica (1e-6) / Remover elementos pequenos (1e-5 - 1m);
6. Identificar Geometrias duplicadas / Identificar Overlaps / Identificar Geometrias inválidas (com correção automática);
7. Ajustar conectividade das linhas (1m de raio) / Adicionar vértices não compartilhados nas intersecções / Adicionar vértices não compartilhados em segmentos compartilhados / Unir linhas / Desagregar geometrias;
8. Identificar Geometrias inválidas (com correção automática) / Identificar ângulos pequenos / Identificar ângulos pequenos entre camadas;
9. Snap Hierárquico;
10. Identificar vértices não compartilhados nas intersecções
11. Identificar vértices não compartilhados nos segmentos compartilhados
12. Verificar elementos viários
13. Verificar intersecções
14. Suavização de Douglas-Peucker / Unir linhas;
15. Identificar Geometrias inválidas (com correção automática) / Identificar vértices próximos de arestas / Identificar vérfice não compartilhado nas intersecções / Identificar vértice não compartilhado em segmentos compartilhados;
16. Identificar geometrias com densidade incorreta de vértices;
17. Identificar Z;
18. Identificar overlaps;
19. Identificar erros de ortografia nos atributos;
20. Identificar erros de atributação.
21. Identificar erros de relacionamentos espaciais;

## Detalhamento dos processos

### 1. Manipulação preliminar de geometrias

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/gerais/manipulacao_preliminar_geometria.model3
- camadas: todas as camadas carregadas;
- processos utilizados: Remover geometrias nulas / Desagregar geometrias / Remover vértices duplicados / Remover feições duplicadas / identify features with invalid unicode;
- black list de atributos: ["id","texto_edicao","label_x","label_y","justificativa_txt","tamanho_txt","visivel","carta_simbolizacao","simbolizar_carta_mini","simb_rot","rotular_carta_mini","espacamento","tamanho_txt","estilo_fonte","cor","cor_buffer","tamanho_buffer","observacao","length_otf","geometry_error","observacao","operador_criacao","data_criacao","operador_atualizacao","data_atualizacao"]
- nome camadas flags: flags_unicode_invalido_ponto,flags_unicode_invalido_linha,flags_unicode_invalido_poligono

### 2. Identifica geometrias inválidas (com correção) e ângulos pequenos

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/gerais/identifica_e_corrige_geometria_invalida_identifica_angulos_pequenos.model3
- processos utilizados: Identificar Geometrias inválidas (com correção automática) / Identificar ângulos pequenos (10 graus);
- camadas: todas as camadas carregadas;
- nome camada flags: flags_geometrias_invalidas
- admite falsos positivos? Não.
- para após a execução? Somente se tiver flags.
- Texto para tooltip: O operador deve corrigir manualmente os apontamentos desse processo.

### 3. Unir linhas com mesmo conjunto de atributos

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/gerais/unir_linhas_com_mesmo_conjunto_de_atributos.model3
- processos utilizados: Unir linhas com mesmo conjunto de atributos
- camada: todas as camadas do tipo linha carregadas;
- nome camada flags: não aponta flags;
- admite falsos positivos? Não é o caso;
- para após a execução? Não.
- black list de atributos: ["id","texto_edicao","label_x","label_y","justificativa_txt","tamanho_txt","visivel","carta_simbolizacao","simbolizar_carta_mini","simb_rot","rotular_carta_mini","espacamento","tamanho_txt","estilo_fonte","cor","cor_buffer","tamanho_buffer","observacao","length_otf","geometry_error","observacao","operador_criacao","data_criacao","operador_atualizacao","data_atualizacao"]
- Texto para tooltip: O algoritmo une linhas com mesmo conjunto de atributos.

### 4. Identificar linhas entrelaçadas

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/via_deslocamento/identifica_rodovias_entrelacadas.model3
- processos utilizados: Identify Intertwined Lines;
- camada: infra_via_deslocamento_l;
- nome camada flags: flags_linhas_entrelacadas;
- admite falsos positivos? Não;
- para após a execução? Somente se tiver flags;
- Texto para tooltip: O operador deve corrigir manualmente linhas que se entrelaçam. Normalmente, tais problemas são de digitalização.
  
### 5. Limpeza Suave das Linhas

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/gerais/limpeza_suave_linhas.model3
- processos utilizados: Clean geometries (1e-6) / Remove small lines (1e-5) / Remove Duplicated Features;
- camada: todas as linhas;
- nome camada flags: não há;
- admite falsos positivos? Não é o caso;
- nome da camada de saída: saida_clean_flags
- para após a execução? Sim
- Texto para tooltip: 
  
### 6. Identifica problemas de construção entre geometrias

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/gerais/identifica_problemas_construcao_entre_geometrias.model3
- processos utilizados: Identificar Geometrias duplicadas / Identificar overlaps / Identificar Geometrias inválidas (com correção automática)
- obs: fluxo genérico para atender diversas etapas de produção (atende os casos de ponto, linha e polígono)
- camada: todas as camadas;
- nome camada flags: flags_p, flags_l, flags_a
  
### 7. Corrige compartilhamento de vértices entre camadas

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/gerais/corrige_compartilhamento_de_vertices.model3
- processos utilizados: Ajustar conectividade das linhas (1m de raio) / Adicionar vértices não compartilhados nas intersecções / Adicionar vértices não compartilhados em segmentos compartilhados / Unir linhas / Desagregar geometrias
- obs: fluxo genérico para atender diversas etapas de produção (atende os casos de ponto, linha e polígono)
- camada: todas as camadas;
- nome camada flags: não é o caso

### 8. Identificar geometrias inválidas e ângulos pequenos entre camadas pós correção de vértices

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/gerais/identifica_e_corrige_geometria_invalida_identifica_angulos_pequenos.model3
- processos utilizados: Identificar Geometrias inválidas (com correção automática) / Identificar ângulos pequenos (10 graus) / Identificar ângulos pequenos entre camadas;
- camadas: todas as camadas carregadas;
- nome camada flags: flags_geometrias_invalidas
- admite falsos positivos? Não.
- para após a execução? Somente se tiver flags.
- Texto para tooltip: O operador deve corrigir manualmente os apontamentos desse processo.

### 9. Snap Hierárquico

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/via_deslocamento/snap_hierarquico_via_deslocamento.model3
- processos utilizados: Snap Hierárquico
- configuração do snap hierárquico: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/snap_hierarquico_via_deslocamento.json

### 10. Identificar vértices não compartilhados nas intersecções

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/interseccao/identifica_vertice_nao_compartilhado_nas_interseccoes.model3
- processos utilizados: Identify Unshared Vertex on Intersections / Extract by location;
- camadas: todas as camadas linha;
- nome camada flags: flag_vertice_nao_compartilhado
- admite falsos positivos? Não.
- para após a execução? Somente se tiver flags.
- Texto para tooltip: Identifica as interseções entre as feições que não possuem um vértice compartilhado nas camadas selecionadas. O operador deve corrigir manualmente os apontamentos desse processo.
  
### 11. Identificar vértices não compartilhados nos segmentos compartilhados

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/interseccao/identifica_vertice_nao_compartilhado_nos_segmentos_compartilhados_interseccoes.model3
- processos utilizados: Interseção de linhas / Identify Unshared Vertex on Shared Edges / Extract by location;
- camadas: todas as camadas linha;
- nome camada flags: flag_vertice_nao_compartilhado_em_seg_compartilhado
- admite falsos positivos? Sim.
- para após a execução? Somente se tiver flags.
- Texto para tooltip: Identifica as feições que não possuem vértices compartilhado nas sobreposições das camadas selecionadas. O operador deve corrigir manualmente os apontamentos desse processo.

### 12. Verificar elementos viários

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/interseccao/verifica_elementos_viarios.model3
- processos utilizados: Interseção de linhas / Extract by location;
- camada: elemnat_trecho_drenagem_l / infra_via_deslocamento_l / infra_elemento_viario_p;
- nome camada flags: flag_elemento_viario;
- admite falsos positivos? Sim;
- para após a execução? Somente se tiver flags. 
- Texto para tooltip: Identifica os elementos viários do tipo ponto que não se encontram nas intersecções de drenagem e via deslocamento. O operador deve corrigir manualmente os apontamentos desse processo.

### 13: Verificar intersecções

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/interseccao/verifica_interseccoes.model3
- processos utilizados: Interseção de linhas / Extract by location;
- camada: elemnat_trecho_drenagem_l / infra_via_deslocamento_l / infra_elemento_viario_p / infra_elemento_viario_l / infra_barragem_l;
- nome camada flags: flag_intersecção;
- admite falsos positivos? Sim;
- para após a execução? Somente se tiver flags. 
- Texto para tooltip: Identifica as intersecções de drenagem permanente com via_deslocamento que não possuem elemento viário. O operador deve corrigir manualmente os apontamentos desse processo.

### 14. Simplificação de Douglas-Peucker

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/gerais/simplificacao_linhas.model3
- processos utilizados: Topological Douglas/Unir linhas;
- camada: todas as linhas;
- nome camada flags: não há;
- admite falsos positivos? Não é o caso;
- nome da camada de saída: flags_suavizacao
- para após a execução? Sim
- Texto para tooltip: 
- black list de atributos: ["id","texto_edicao","label_x","label_y","justificativa_txt","tamanho_txt","visivel","carta_simbolizacao","simbolizar_carta_mini","simb_rot","rotular_carta_mini","espacamento","tamanho_txt","estilo_fonte","cor","cor_buffer","tamanho_buffer","observacao","length_otf","geometry_error","observacao","operador_criacao","data_criacao","operador_atualizacao","data_atualizacao"]

### 15: Identifica problemas de compartilhamento de vértices

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/gerais/identifica_problemas_compartilhamento_vertices.model3
- processos utilizados: Identificar Geometrias inválidas (com correção automática) / Identificar vértices próximos de arestas / Identificar vértice não compartilhado nas intersecções / Identificar vértice não compartilhado em segmentos compartilhados;
- nome camada flags: flag_geometrias_invalidas,flag_vertices_proximo_arestas,flag_vertices_nao_compartilhados_interseccoes,flag_vertice_nao_compartilhado_em_seg_compartilhado, flag_linha_nao_seccionada_na_interseccao
- Texto para tooltip: Todas as feições devem compartilhar vértices, logo, onde for apontado erro, deve-se adicionar o vértice nas linhas que possuem intersecção ponto ou linha.

### 16: Identificar geometrias com densidade incorreta de vértices

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/gerais/identifica_geometrias_com_densidade_incorreta_de_vertices.model3
- camadas: todas as camadas carregadas;
- tol: 0.00001 grau
- nome camada flags: flag_densidade_incorreta_vertices

### 17. Identificar Z

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/gerais/identifica_z.model3
- camadas: todas carregadas
- nome camada flags: flag_z

### 18. Identificar overlaps dentro da mesma camada

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/gerais/identifica_overlaps_linhas.model3
- camadas: todas
- nome camada flags: flags_overlaps_l

### 19. Identificar erros de ortografia no atributo nome

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/gerais/identifica_erro_ortografia_atributo_nome.model3;
- camadas: todas;
- para após a execução? Sim
- nome camada de saída: saida_verifica_ortografia_nome

### 20. Identificar erros de atributação

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/gerais/identifica_erros_atributacao.model3
- camadas: todas;
- para após a execução? Sim
- nome camada de flags: flags_erros_atributos
- nome camada de saída: atributos_incomuns

### 21. Identificar erros de relacionamentos espaciais

- arquivo: /configuracoes_producao/edgv_topo/1_3/modelo_qgis/via_deslocamento/identifica_erros_relacionamentos_espaciais_transportes.model3
- camadas: todas;
- nome camada de flags: flags_ponto,flags_linha