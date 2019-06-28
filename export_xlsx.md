[Index](index.html)

# Export to XLSX file

Export data from SQLAlquemy query to and Excel file

```python
@expose()
def export(self, **kw):
    """
    Export products in xlsx format
    :param kw:
    :return:
    """

    columns = [
        (model.Product.model, 'model', 'Clave'),
        (model.Product.code, 'code', 'EAN'),
        (model.Product.name, 'name', 'Descripción'),
        (model.Product.units_per_package, 'units_per_package', 'Unit'),
        (model.Product.tax_type_id, 'tax_type_id', 'Cve'),
        (model.Product.size_depth, 'size_depth', 'Largo'),
        (model.Product.size_width, 'size_width', 'Ancho'),
        (model.Product.size_height, 'size_height', 'Alto'),
        (model.Product.volume, 'volume', 'M3'),
        (model.Product.normal_price, 'normal_price', 'Precio')
    ]

    query = model.DBSession.query(model.Product) \
        .join(model.Brand) \
        .order_by(model.Product.model.asc())

    # add columns to query
    query = query.add_columns(*[c[0] for c in columns])

    column_names = [
        col[1] for col in columns
    ]

    results = [row[1:] for row in query.all()]

    xlsx_file = export_xlsx(column_names, results)

    response.headers["Content-Type"] = "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
    response.headers["Content-disposition"] = "attachment; filename=export.xlsx"
    response.headers["Content-Length"] = str(len(xlsx_file))
    
    return xlsx_file

```

Using openpyxl

```python
# -*- coding: utf-8 -*-

from openpyxl import Workbook
from openpyxl.writer.excel import save_virtual_workbook

def export_xlsx(fields, data):
    """
    Export simple table to xlsx file
    :param fields: comma separated field names
    :param data: comma rows in field order
    :return: xlsx bytes stream
    """

    wb = Workbook()

    # grab the active worksheet
    ws = wb.active
    ws.title = "product"

    # header columns
    ws.append(fields)

    for row in data:
        ws.append(row)

    # openpyxl provides a function to save a file to a StringIO object
    stream = save_virtual_workbook(wb)

    return stream

```
