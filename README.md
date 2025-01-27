# prova

# Inizializza la colonna per il prelievo
df_cuso_excess_adjustment["WITHDRAWN_AMOUNT"] = 0

# Raggruppa per SCENARIO_ID e AGREEMENT_ID
grouped = df_cuso_excess_adjustment.groupby(["SCENARIO_ID", "AGREEMENT_ID"])

# Applica la logica gruppo per gruppo
for (scenario_id, agreement_id), group in grouped:
    margin_to_cover = -group["EXCESS_MARGIN"].iloc[0]  # Il valore totale da coprire (positivo o 0)

    # Se l'EXCESS_MARGIN è >= 0, nessun prelievo è necessario
    if margin_to_cover <= 0:
        continue  # Salta al prossimo gruppo

    for i in group.index:  # Itera sulle righe del gruppo
        # Valori correnti
        unstress_mtm_value = -df_cuso_excess_adjustment.loc[i, "EMTM_VALUE_UNSTRESS"]  # Disponibile (positivo)

        # Calcola quanto prelevare
        if margin_to_cover > unstress_mtm_value:
            withdrawal = unstress_mtm_value  # Preleva tutto il collaterale della riga
        else:
            withdrawal = margin_to_cover  # Preleva solo quanto necessario

        # Aggiorna il prelievo nella riga corrente
        df_cuso_excess_adjustment.loc[i, "WITHDRAWN_AMOUNT"] = withdrawal

        # Aggiorna il margine residuo
        margin_to_cover -= withdrawal

        # Interrompi il ciclo se il margine è completamente coperto
        if margin_to_cover <= 0:
            break
