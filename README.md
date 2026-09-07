# Odoo

## Implémentation SaaS Studio/QWeb — Tableau « Origine matériel »

### 1) Vues QWeb à modifier
- **Facture client (PDF)** : `account.report_invoice_document`
- **Bon de livraison (PDF)** : `stock.report_delivery_document`

### 2) Snippets QWeb prêts à coller (héritage Studio)

```xml
<odoo>
  <!-- FACTURE CLIENT -->
  <template id="x_origin_material_invoice" inherit_id="account.report_invoice_document">
    <!-- Insertion juste avant le bloc des totaux -->
    <xpath expr="//div[@id='total']" position="before">
      <t t-set="origin_products"
         t-value="o.invoice_line_ids.filtered(lambda l: l.product_id and not l.display_type).mapped('product_id')"/>
      <t t-if="origin_products">
        <h4>Origine matériel</h4>
        <table class="table table-sm o_main_table">
          <thead>
            <tr>
              <th>Produit</th>
              <th class="text-end">Quantité</th>
              <th>Code origine</th>
              <th>Pays d’origine</th>
            </tr>
          </thead>
          <tbody>
            <t t-foreach="origin_products" t-as="p">
              <t t-set="product_lines"
                 t-value="o.invoice_line_ids.filtered(lambda l: l.product_id == p and not l.display_type)"/>
              <tr>
                <td><span t-field="p.display_name"/></td>
                <td class="text-end"><span t-esc="sum(product_lines.mapped('quantity'))"/></td>
                <td>
                  <span t-esc="
                    (p.product_tmpl_id._fields.get('x_studio_code_origine') and p.product_tmpl_id['x_studio_code_origine'])
                    or (p.product_tmpl_id._fields.get('x_code_origine') and p.product_tmpl_id['x_code_origine'])
                    or (p.product_tmpl_id._fields.get('origin_code') and p.product_tmpl_id['origin_code'])
                    or (p._fields.get('x_studio_code_origine') and p['x_studio_code_origine'])
                    or (p._fields.get('x_code_origine') and p['x_code_origine'])
                    or (p._fields.get('origin_code') and p['origin_code'])
                    or ''
                  "/>
                </td>
                <td>
                  <span t-esc="
                    (p.product_tmpl_id._fields.get('country_of_origin_id') and p.product_tmpl_id['country_of_origin_id'] and p.product_tmpl_id['country_of_origin_id'].name)
                    or (p.product_tmpl_id._fields.get('x_studio_pays_origine') and p.product_tmpl_id['x_studio_pays_origine'])
                    or (p.product_tmpl_id._fields.get('x_pays_origine') and p.product_tmpl_id['x_pays_origine'])
                    or (p._fields.get('country_of_origin_id') and p['country_of_origin_id'] and p['country_of_origin_id'].name)
                    or (p._fields.get('x_studio_pays_origine') and p['x_studio_pays_origine'])
                    or (p._fields.get('x_pays_origine') and p['x_pays_origine'])
                    or ''
                  "/>
                </td>
              </tr>
            </t>
          </tbody>
        </table>
      </t>
    </xpath>
  </template>

  <!-- BON DE LIVRAISON -->
  <template id="x_origin_material_delivery" inherit_id="stock.report_delivery_document">
    <!-- Insertion après le tableau des lignes produits -->
    <xpath expr="//table[@name='stock_move_line_table']" position="after">
      <t t-set="origin_products"
         t-value="o.move_ids_without_package.filtered(lambda m: m.product_id and (not m._fields.get('display_type') or not m.display_type)).mapped('product_id')"/>
      <t t-if="origin_products">
        <h4>Origine matériel</h4>
        <table class="table table-sm">
          <thead>
            <tr>
              <th>Produit</th>
              <th class="text-end">Quantité</th>
              <th>Code origine</th>
              <th>Pays d’origine</th>
            </tr>
          </thead>
          <tbody>
            <t t-foreach="origin_products" t-as="p">
              <t t-set="product_moves"
                 t-value="o.move_ids_without_package.filtered(lambda m: m.product_id == p and (not m._fields.get('display_type') or not m.display_type))"/>
              <t t-set="qty_total" t-value="0"/>
              <t t-foreach="product_moves" t-as="m">
                <t t-set="qty_total" t-value="qty_total + (m.quantity_done or m.product_uom_qty or 0)"/>
              </t>
              <tr>
                <td><span t-field="p.display_name"/></td>
                <td class="text-end"><span t-esc="qty_total"/></td>
                <td>
                  <span t-esc="
                    (p.product_tmpl_id._fields.get('x_studio_code_origine') and p.product_tmpl_id['x_studio_code_origine'])
                    or (p.product_tmpl_id._fields.get('x_code_origine') and p.product_tmpl_id['x_code_origine'])
                    or (p.product_tmpl_id._fields.get('origin_code') and p.product_tmpl_id['origin_code'])
                    or (p._fields.get('x_studio_code_origine') and p['x_studio_code_origine'])
                    or (p._fields.get('x_code_origine') and p['x_code_origine'])
                    or (p._fields.get('origin_code') and p['origin_code'])
                    or ''
                  "/>
                </td>
                <td>
                  <span t-esc="
                    (p.product_tmpl_id._fields.get('country_of_origin_id') and p.product_tmpl_id['country_of_origin_id'] and p.product_tmpl_id['country_of_origin_id'].name)
                    or (p.product_tmpl_id._fields.get('x_studio_pays_origine') and p.product_tmpl_id['x_studio_pays_origine'])
                    or (p.product_tmpl_id._fields.get('x_pays_origine') and p.product_tmpl_id['x_pays_origine'])
                    or (p._fields.get('country_of_origin_id') and p['country_of_origin_id'] and p['country_of_origin_id'].name)
                    or (p._fields.get('x_studio_pays_origine') and p['x_studio_pays_origine'])
                    or (p._fields.get('x_pays_origine') and p['x_pays_origine'])
                    or ''
                  "/>
                </td>
              </tr>
            </t>
          </tbody>
        </table>
      </t>
    </xpath>

    <!-- Fallback d’ancrage si la table précédente diffère selon version -->
    <xpath expr="//table[@name='stock_move_table']" position="after">
      <t t-set="origin_products"
         t-value="o.move_ids_without_package.filtered(lambda m: m.product_id and (not m._fields.get('display_type') or not m.display_type)).mapped('product_id')"/>
      <t t-if="origin_products and not 0">
        <!-- duplicat volontairement neutralisé; garder uniquement un des 2 xpaths actif en Studio -->
      </t>
    </xpath>
  </template>
</odoo>
```

### 3) Emplacement exact d’insertion
- **Facture**: dans l’héritage de `account.report_invoice_document`, insérer le bloc via xpath **avant** `//div[@id='total']`.
- **Bon de livraison**: dans l’héritage de `stock.report_delivery_document`, insérer le bloc **après** `//table[@name='stock_move_line_table']`.
  - Si votre version n’a pas `stock_move_line_table`, ancrer **après** `//table[@name='stock_move_table']`.

### 4) Fallback robuste champ “code origine”
Le snippet teste plusieurs champs, dans cet ordre, sans casser le rendu si absent :
- Template produit: `x_studio_code_origine` → `x_code_origine` → `origin_code`
- Variante produit: `x_studio_code_origine` → `x_code_origine` → `origin_code`
- Sinon chaîne vide.

Le pays d’origine suit la même logique (`country_of_origin_id`, puis variantes Studio/custom), sinon vide.

### 5) Plan de tests (minimum)
1. **Produit dupliqué sur facture**
   - Créer 2 lignes facture avec le même produit (quantités 2 et 3).
   - Attendu PDF: une seule ligne “Origine matériel” pour ce produit, quantité = 5.
2. **Origine manquante**
   - Produit sans code origine ni pays d’origine.
   - Attendu PDF: colonnes “Code origine” et “Pays d’origine” vides, sans erreur.
3. **Bon de livraison multi-lignes**
   - BL avec le même produit sur plusieurs mouvements.
   - Attendu PDF: regroupement par produit, quantité sommée.
4. **Lignes section/note (facture)**
   - Ajouter section et note dans les lignes.
   - Attendu: non affichées dans le tableau “Origine matériel”.

### Rollback simple
- Aller dans **Studio > Rapports**.
- Désactiver (ou archiver/supprimer) les vues héritées Studio ajoutées sur:
  - `account.report_invoice_document`
  - `stock.report_delivery_document`
- Réimprimer les PDF: retour immédiat au rendu standard.
