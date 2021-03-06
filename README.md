Tastyworks PNL
--------------

Create a PnL-document for a tax income statement in Germany for the US-broker tastyworks.
Download all transactions from Tastyworks into a csv-file and create with this
python-script a new enhanced csv-file. This new csv-file adds a currency conversion
from USD to Euro and computes the realised profits and losses based on the FIFO method.
Also all currency gains for the USD-cash-account are added.
A summary for all stock, dividend, option gains as well as the new "Termingeschäftsverluste"
is output.

Details on German taxes are written up
at <https://laroche.github.io/private-geldanlage/steuern.html>.

A web-based application will be built up at <https://knorke2.homedns.org/depot-pnl>.


[In German: Erstellt eine Gewinn- und Verlustberechnung für eine Steuererklärung
in Deutschland für den US-Broker Tastyworks.
Lade von Tastyworks alle Transaktionen in eine CSV-Datei herunter und generiere
mit diesem Python-Skript eine neu aufbereitete CSV-Datei. Diese neue CSV-Datei
fügt eine Umrechnung von USD in Euro hinzu und berechnet die realisierten
Gewinne und Verluste nach FIFO-Methode. Auch eine Berechnung aller Währungsgewinne
für das USD-Cashkonto wird erstellt.
Eine Übersicht über die verschiedenen Summen für Aktien/Dividenden/Optionen
und den neuen Topf der Termingeschäftsverluste wird ausgegeben.

Details zur Steuererklärung in Deutschland sind unter
<https://laroche.github.io/private-geldanlage/steuern.html> zusammengestellt.

Eine Web-Applikation für dieses Python-Skript wird in Zukunft
unter <https://knorke2.homedns.org/depot-pnl> aufgebaut.]


How to use
----------

Download your trade history as csv file from
<https://trade.tastyworks.com/index.html#/transactionHistoryPage>.
(Choose "Activity" and then "History" and then setup the filter for a
custom period of time and download it as csv file. Do not download this file
from the separate Tastyworks application, but use the website and your browser.)

Newest entries in the csv file should be on the top and it should contain the complete
history over all years. (You can at most download data for one year, so for several years
you have to download into several files and combine them into one file with a text editor.)
The csv file has the following first line:

<code>
Date/Time,Transaction Code,Transaction Subcode,Symbol,Buy/Sell,Open/Close,Quantity,Expiration Date,Strike,Call/Put,Price,Fees,Amount,Description,Account Reference
</code>

If you delete the __eurusd.csv__ file, a current version is downloaded directly
from <https://www.bundesbank.de/de/statistiken/wechselkurse>.
(Link to the data: [eurusd.csv](https://www.bundesbank.de/statistic-rmi/StatisticDownload?tsId=BBEX3.D.USD.EUR.BB.AC.000&its_csvFormat=en&its_fileFormat=csv&mode=its&its_from=2010))

The option __--usd__ can be used to not translate pnl data into Euro.

The option __--verbose__ adds currency gains for each transaction.

The option __-b__ adds all losses from option writing into __term_loss__. This is needed to compute
all non-stock losses for the German tax statement.

Per default balance adjustments are only output as total sum, you can use the option __--long__
to include balance adjustments into the list of transactions.

Per default the script stops on unknown trading symbols (underlyings) and you have
to hardcode into the source code if it is an individual stock or some ETF/fond.
You can use the __--assume-individual-stock__ option to assume individual stock for all unknown symbols.

Currency gains are computed via FIFO as with other assets. They are tax free if hold for more than one year
or if credit is paid back (negative cash balance). They are also tax free for dividends, credit payments and
sold options as well as account fees.
Currency gains need to go into the 'Anlage SO' within a private German tax statement.

The summary output lists all assets at the end of each year. 'account-usd' contains a FIFO list of all
USD buys and the conversion price for Euro. This entry might be very long and look complicated. You might
want to ignore this line and just look over the list of other assets.

The option __--debug-fifo__ gives details on the FIFO workings. Be aware that pnl data
is the cummulative sum, not the real local change. (Bug in the output!)

The option __--show__ gives some summary graphs.


If you work on Linux with Ubuntu/Debian, you need to make sure
<https://pandas.pydata.org/> is installed:

<code>
sudo apt-get install python3-pandas
</code>

Or use pip for a local install:

<code>
pip install pandas
</code>


CSV and Excel Output
--------------------

The script can also output all data again as CSV file or as Excel file.
(CSV should be most robust, I don't have much experience with excel. I'd recommend CSV
and just reading it into a new Excel sheet yourself. Both data types contain the same output data.)

The options for this are __--output-csv=file.csv__ and __--output-excel=file.xlsx__.

The output contains the important original data from the Tastyworks csv file plus
pnl generated data as well as eurusd conversion data. You probably do not have to
provide all data in a tax statement, some is only added for further data processing
convenience in your spreadsheet program.

If you provide the option __--tax-output=2020__, your output file will be
optimized for tax output for a special year. Only transactions for that year are output.
The datetime will only contain the day, but no time information. Fewer other data is output.

The CSV file contains floating point numbers in English(US) notation where a decimal
point (".") and no decimal comma (",") is used as decimal separator for floating point
numbers like in "-0.2155".
Usually Tastyworks provides floating point numbers with 4 decimals after the decimal separator.
For yearly summaries I only output with the normal 2 decimal digits.

Here the output transaction data in detail:

- __datetime__: Date and time (Tastyworks gives minutes for this, no exact seconds)
  of the transaction
- __type__: type of transaction (German names): Aktie, Aktienfond, Mischfond,
  Immobilienfond, Sonstiges, Option, Future, Dividende, Quellensteuer, Zinsen,
  Ein/Auszahlung, Ordergebühr, Brokergebühr
- __pnl__: pnl for tax payments for this transaction based on FIFO
- __usd_gains__: currency conversion gains for the account in USD. Based on cash changes
  in USD due to this transaction.
- __usd_gains_notax__: as above, but not part of German tax law
- __eur_amount__: 'amount - fees' converted into Euro currency
- __amount__: transaction amount in USD
- __fees__: cost of transaction in USD that needs to be subtracted from amount (Tastyworks also pays commissions, but they are not listed in the trade history)
- __eurusd__: official eurusd conversion rate for this transaction date from bundesbank.de
- __quantity__: number of buys or sells
- __asset__: what is bought (stock symbol or something like 'SPY P310 20-12-18' for an option
- __symbol__: base asset (underlying) that is traded. This is included to be able to
  generate summary overviews for e.g. all transactions in SPY with stocks and options combined.
- __description__: additional informational text for the transaction
- __account_total__: account cash balance in USD after this transaction. This is the previous account total plus
  'amount - fees' from this transaction. (Cash amount at Tastyworks.)
  This is purely informational and not needed for tax data.
- __net_total__: Sum in USD of account_total (cash) plus all assets (stocks, options)
  in your account.
  This does not use current market data, but keeps asset prices at purchase cost.
  Best looked at to check if this script calculates the same total sum as shown in your
  Tastyworks current total.
- __term_loss__: how much does this transaction contribute to losses in future
  contracts ('Verlustverrechnungstopf Termingeschäfte'). If you use the option __-b__ then
  also all losses from option writing are added here.
- __tax_free__: are further currency changes tax free (German: steuerneutral)


FAQ
---

- Maybe ACH transfers are not yet implemented. I don't use them, maybe email me
  a sample transaction line, so that I can adjust the source code.
- Either github issues or email works for me to enhance/fix this program. Sample data
  is best to resolve issues.
- One check is to look at the computed value of "account cash balance" from this script and compare it
  to the official value "Cash Balance" from Tastyworks at <https://manage.tastyworks.com/#/accounts/balances>.
- The output-csv file should have the same amount of lines as the input csv. The resulting output
  is just an enriched form of the input. Please verify all summary output by computing it again from
  the output-csv file yourself.


Currency gains in German tax law
--------------------------------

This description has moved to <https://laroche.github.io/private-geldanlage/steuern.html#fremdwaehrungskonten>.


TODO
----

Important:

- If a long option is assigned, the option buy price should be added to
  the stock price. This is currently not done, but we print a warning
  message for this case for manual adjustments in this rather rare case.
- If option writing is cash-settled, the cash-settlement needs to go into "Termingeschäftsverluste".
- Currently we only make one pass over the data. Better allow several data passes/computations.
- Print header with explanation of transaction output.
- Can Excel output also include yearly summary data computed from Excel?
  Can transactions also be grouped per year on different sheets?
- Optionally break up report into: dividends, withholding-tax, interest, fees, stocks, other.
  (We now have a "type" field doing this, just filter your csv data on this.)
- Does not work with futures.
   - A first quick implementation is done, further checks are needed on real account data.
     Let me know if you can provide full account data to check/verify.
   - As "Mark-to-Market"-payments/transactions are done, we immediately pay taxes for them, this is
     not stacked up until futures are sold again. What is a correct way to tax these transactions?
     Adding this up needs to be done to correctly implement "20k€ Terminverlustgeschäfte" for futures.
- Add extra counter for all transactions to/from the account.
- Stock splits and spinoffs are not fully supported. (Option strike prices also need to be adjusted.) Example entry:
<pre>
01/21/2021 12:39 PM,Receive Deliver,Forward Split,TQQQ,Buy,Open,108,,,,,0.00,-9940.86,Forward split: Open 108.0 TQQQ,xxx...00
01/21/2021 12:39 PM,Receive Deliver,Forward Split,TQQQ,Sell,Close,54,,,,,0.00,9940.86,Forward split: Close 54.0 TQQQ,xxx...00
</pre>
   - Assumption: stock/option splits are tax neutral.
   - stock splits are now implemented, but not tested at all. Options are not yet supported. Please send in more test data.
- For currency gains, we could also add all fees as tax free by adding a separate booking/transaction.
- For currency gains tax calculation you can reorder all transactions of one day and use the best
  order to minimize tax payments. This is currently not done with the current source code.
- If you transfer USD to another bank account, you need to choose between tax-neutral and normal tax transaction.
- In German: Stillhalterpraemien gelten auch nicht als Währungsanschaffung, sondern
  als Zufluss und sind daher steuer-neutral. Im Source wird dazu die Auszeichnung von Tastyworks
  als "Sell-To-Open" verwendet. Was passiert aber, wenn man eine Option gekauft hat und dann 2 davon
  verkauft? Bleibt das bei Tastyworks eine Transaktion oder finden hier dann zwei Transaktionen statt?
  Dieser Fall tritt bei mir nicht auf. Der Source Code sollte zumindest diesen Fall detektieren und
  eine Warnung ausgeben.
- Add counters for all stock losses and non-stock losses for German tax report.
- In German: Bei Aktien-Leerverkäufen (über eine Jahresgrenze hinaus) wird 30 % vom Preis mit der KapESt
  als Ersatzbemessungsgrundlage besteuert (§ 43a Absatz 2 Satz 7 EStG) und erst mit der Eindeckung ausgeglichen.
- Complete support for Investmentsteuergesetz (InvStG) 2018.
   - German: Teilfreistellungen gelten auch für Dividenden
- Add images on how to download csv-file within Tastyworks into docu.
- Complete the list of non-stocks.
- Done: For an individual stock whitelist we could list all SP500 and Nasdaq100 stocks.
  How do we cope with historical data for this?
- Specify non-realised gains to know how much tax needs to be paid for current net total.
- Add performance reviews, graphs based on different time periods and underlyings.
- Add description of the asset: SPY: SPDR S&P 500 ETF Trust
- Check if withholding tax is max 15% for US stocks as per DBA.
  Warn if e.g. 30% withholding tax is paid and point to missing W8-BEN formular.
- Are we rounding output correctly?
- Is the time output using the correct timezone?
- Does Tastyworks use BRK.B or BRK/B in transaction history?
  Adjust the list of individual stocks accordingly.
- Look at other libraries for currency conversion:
  <https://github.com/alexprengere/currencyconverter> or
  <https://github.com/flaxandteal/moneypandas>
- Generate documentation that can be passed on to the German tax authorities on how
  the tax is computed and also document on how to fill out the official tax documents.
   - Check this for docx reporting: <https://github.com/airens/interactive_brokers_tax>

Nice:

- Translate text output into German.
- Add docu in German.
- Add test data for users to try out.
- Add testsuite to verify proper operation.
- Improve output of open positions.
- Use pandas.isna(x)?

