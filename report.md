i. Which insights did you gain from your EDA?

The most significant finding was severe class imbalance: only 0.1291% of transactions are fraudulent, meaning accuracy alone would be a misleading metric and F1 score was necessary from the start. Fraud also concentrates almost entirely in two transaction types — TRANSFER (~0.77% fraud rate) and CASH_OUT (~0.18% fraud rate), while PAYMENT, CASH_IN, and DEBIT contain essentially no fraud. The amount column was heavily right skewed, requiring a log1p transform to reveal its true distribution; after transforming, fraudulent transactions showed a notably higher average amount. Also a particularly strong though rare pattern emerged,investigating 16 transactions with amount == 0: all 16 were CASH_OUT and all 16 were fraud, with the origin account balance at exactly $0 both before and after a 100% fraud rate within a very tiny subset, a notable pattern flagged as a potential engineered feature.

ii. How did you determine which columns to drop or keep?

nameOrig and nameDest were dropped because they are unique identifiers with no repeatable predictive value;keeping them would add noise and dimensionality without helping the model generalize. Outliers were deliberately not removed, since EDA showed fraud transactions are themselves the extreme, unusual cases (larger amounts, larger balance drains); stripping them would remove the exact signal needed to detect fraud. Two new features were engineered directly from EDA findings: orig_balance_delta (oldbalanceOrg - newbalanceOrig), based on the bivariate finding that fraud consistently drains the origin account by a large amount, and is_zero_amt_cashout, based on the 100% fraud rate pattern found in the zero amount CASH_OUT subset. Type was one hot encoded since it was confirmed as a strong categorical predictor, with fraud concentrated almost entirely in TRANSFER and CASH_OUT.

iii. Which hyperparameter tuning strategy did you use? Why?

RandomizedSearchCV was used instead of GridSearchCV because testing every possible combination with an exhaustive GridSearchCV would have taken far too long given the size of the dataset (6.36 million rows). RandomizedSearchCV tests only a smaller, randomly chosen set of combinations instead of all of them; in this case, n_estimators values of 50 and 75, and max_depth values of 8 and 12 and found {'n_estimators': 75, 'max_depth': 12} to be the best combination. This gave similar tuning benefit in far less time than an exhaustive search would have needed.

iv. How did your model's performance change after tuning?

After tuning, the model settled on 75 trees with a max depth of 12, notably smaller and shallower than the untuned baseline's default settings (100 trees, unlimited depth). Despite using less complexity, the tuned model achieved a fraud class F1 score of 0.8424, with precision of 0.98 and recall of 0.74. This suggests the extra depth and tree count in the baseline weren't adding meaningful predictive value for this problem, and a leaner model performed just as well.

v. Final F1 score and precision vs. recall interpretation

The final F1 score was 0.8424. The precision of 0.98 means that when the model predicts a transaction is fraudulent, it is correct 98% of the time, very few legitimate customers would be wrongly flagged, which matters since false fraud alerts create customer friction and unnecessary required customer service. The recall of 0.74 means the model successfully identifies about 74% of all actual fraud cases, leaving roughly 26% of fraud undetected. This precision heavy tradeoff suggests the model is conservative: it only flags transactions it's highly confident about, which minimizes false alarms at the cost of missing a meaningful share of real fraud. This tradeoff would need to be weighed against business priorities banks often favor higher recall (catching more fraud, tolerating more false alarms) since missed fraud is typically more costly.

