Clustering et identification des similarités : Détails et code
Voici une explication plus précise du processus de clustering et d'identification des similarités, ainsi que les parties du code concernées :
1. Calcul des embeddings avec Sentence Transformers
Le code utilise la bibliothèque Sentence Transformers pour calculer les embeddings des phrases de contenu. Les embeddings sont des représentations vectorielles des phrases, où des phrases similaires auront des vecteurs proches dans l'espace vectoriel.
La fonction model.encode() est utilisée pour calculer les embeddings. Elle prend en entrée une liste de phrases et renvoie une liste de vecteurs.
corpus_embeddings = model.encode(corpus_sentences, batch_size=256, show_progress_bar=True, convert_to_tensor=True)
Use code with caution.
Python
2. Détection de communauté pour le clustering
La détection de communauté est une technique pour identifier des groupes (clusters) d'éléments similaires dans un graphe. Dans ce cas, le graphe est implicite, où les nœuds sont les phrases et les arêtes représentent la similarité entre les phrases.
La fonction util.community_detection() de Sentence Transformers est utilisée pour effectuer la détection de communauté. Elle prend en entrée les embeddings des phrases et renvoie une liste de clusters.
clusters = util.community_detection(corpus_embeddings, min_community_size=2, threshold=min_similarity)
Use code with caution.
Python
3. Association des URLs aux clusters
Le script parcourt les clusters et pour chaque phrase dans un cluster, il récupère l'URL correspondante dans le DataFrame original.
Une nouvelle table est créée avec les colonnes source_h1 (nom du cluster), kw_col (contenu de la phrase) et Address (URL).
for keyword, cluster in enumerate(clusters):
    for sentence_id in cluster[0:]:
        corpus_sentences_list.append(corpus_sentences[sentence_id])
        cluster_name_list.append("Cluster {}, #{} Elements ".format(keyword + 1, len(cluster)))
df_new = pd.DataFrame(None)
df_new['source_h1'] = cluster_name_list
df_new[kw_col] = corpus_sentences_list
df_all.append(df_new)
Use code with caution.
Python
4. Renommage des clusters et fusion des données
Le script renomme les clusters en utilisant la phrase la plus courte du cluster.
Les données originales du crawl sont fusionnées avec les informations de clustering pour obtenir une table avec les URLs source, les titres source, les URLs de destination et les titres de destination.
# Renommage des clusters
df['length'] = df[kw_col].astype(str).map(len)
df = df.sort_values(by="length", ascending=True)
df['source_h1'] = df.groupby('source_h1')[kw_col].transform('first')

# Fusion des données
df2 = df[["Address", kw_col]].copy()
df2.rename(columns={"Address": "source_url", kw_col: "source_h1"}, inplace=True)
df = df.merge(df2.drop_duplicates('source_h1'), how='left', on="source_h1")
Use code with caution.
Python
En résumé, le script utilise Sentence Transformers pour transformer les phrases de contenu en vecteurs, puis la détection de communauté pour regrouper les phrases similaires. Les URLs sont ensuite associées aux clusters correspondants pour identifier les opportunités d'interlinking.
