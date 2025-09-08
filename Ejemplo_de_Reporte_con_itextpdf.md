# Reporte de Ingesos usando libreria itextpdf
Indrodución

## 1. Instalación de dependencia itextpdf
Editar el archivo pom.xml y agregar dependencia, como se muestra a continucaión 
```xml
<dependency>
      <groupId>com.itextpdf</groupId>
			<artifactId>itextpdf</artifactId>
			<version>5.5.13.4</version>
</dependency>
```
## 2. Crear DTO para el resultado de la consulta

```Java
package com.devsoft.orders_api.dto;

import lombok.AllArgsConstructor;
import lombok.Getter;
import java.math.BigDecimal;
import java.time.LocalDate;

@Getter
@AllArgsConstructor
public class IngresoDTO {
    private Long ordenId;
    private String correlativo;
    private LocalDate fechaPago;
    private BigDecimal totalOrden;
    private String clienteNombre;
    private Integer mesaNumero;
    private String meseroNombre;
    private BigDecimal sumaPagos;
}

```
## 3. Crear método con query para obtener los resultados del reporte en el respositorio PagoRepository
```Java
@Query("SELECT new com.devsoft.orders_api.dto.IngresoDTO(" +
            "o.id, o.correlativo, p.fechaPago, o.total, c.nombre, m.numero, u.nombre, SUM(p.monto)) " +
            "FROM Pago p " +
            "JOIN p.orden o " +
            "JOIN o.cliente c " +
            "JOIN o.mesa m " +
            "JOIN o.usuario u " +
            "WHERE p.fechaPago BETWEEN :fechaInicio AND :fechaFin " +
            "GROUP BY o.id, o.correlativo, p.fechaPago, o.total, c.nombre, m.numero, u.nombre " +
            "ORDER BY p.fechaPago ASC")
    List<IngresoDTO> obtenerIngresosRangoFechas(
            @Param("fechaInicio") LocalDate fechaInicio,
            @Param("fechaFin") LocalDate fechaFin);
```
## 4. Programar service que arma los datos y genera PDF (iText)
Aquí es donde se construye el reporte con los datos que se obtienen del repositorio, usando clases y elementos de iTextPdf

```Java
@Service
@RequiredArgsConstructor
public class ReporteService {
    @Autowired
    private PagoRepository pagoRepository;

    //método que genera el pdf y devuelve los bytes
    public byte[] generarReporte(LocalDate inicio, LocalDate fin, boolean detallado) throws Exception {
        //obtenemos los datos para el reporte
        List<IngresoDTO> data = pagoRepository.obtenerIngresosRangoFechas(inicio, fin);

        Document document = new Document(PageSize.LETTER, 36, 36, 72, 36); // márgenes
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        PdfWriter writer = PdfWriter.getInstance(document, baos);

        // Evento para numeración de páginas
        writer.setPageEvent(new PdfPageEventHelper() {
            @Override
            public void onEndPage(PdfWriter writer, Document document) {
                ColumnText.showTextAligned(
                        writer.getDirectContent(),
                        Element.ALIGN_CENTER,
                        new Phrase(String.format("Página %d", writer.getPageNumber()),
                                FontFactory.getFont(FontFactory.HELVETICA, 9, Font.ITALIC)),
                        (document.right() - document.left()) / 2 + document.leftMargin(),
                        document.bottom() - 20,
                        0
                );
            }
        });

        document.open();

        // ========= Encabezado =========
        PdfPTable header = new PdfPTable(2);
        header.setWidthPercentage(100);
        header.setWidths(new int[]{1, 4});

        try {
            Image logo = Image.getInstance("src/main/resources/static/logo.jpg");
            logo.scaleToFit(70, 70);
            PdfPCell logoCell = new PdfPCell(logo, false);
            logoCell.setBorder(Rectangle.NO_BORDER);
            logoCell.setHorizontalAlignment(Element.ALIGN_LEFT);
            header.addCell(logoCell);
        } catch (Exception e) {
            PdfPCell emptyCell = new PdfPCell(new Phrase(" "));
            emptyCell.setBorder(Rectangle.NO_BORDER);
            header.addCell(emptyCell);
        }

        PdfPCell datosEmpresa = new PdfPCell();
        datosEmpresa.setBorder(Rectangle.NO_BORDER);
        datosEmpresa.addElement(new Paragraph("Mi Restaurante S.A. de C.V",
                FontFactory.getFont(FontFactory.HELVETICA_BOLD, 14)));
        datosEmpresa.addElement(new Paragraph("Dirección: Bario El Centro, Chalatenango",
                FontFactory.getFont(FontFactory.HELVETICA, 10)));
        datosEmpresa.addElement(new Paragraph("Tel: 2301-3333",
                FontFactory.getFont(FontFactory.HELVETICA, 10)));
        header.addCell(datosEmpresa);

        document.add(header);
        document.add(new Paragraph(" "));

        Paragraph titulo = new Paragraph("Reporte de Ingresos",
                FontFactory.getFont(FontFactory.HELVETICA_BOLD, 16));
        titulo.setAlignment(Element.ALIGN_CENTER);
        document.add(titulo);

        Paragraph rango = new Paragraph("Del " + inicio + " al " + fin,
                FontFactory.getFont(FontFactory.HELVETICA, 11));
        rango.setAlignment(Element.ALIGN_CENTER);
        document.add(rango);
        document.add(new Paragraph(" "));

        // ========= Contenido =========
        if (detallado) {
            PdfPTable table = new PdfPTable(7);
            table.setWidthPercentage(100);
            table.setWidths(new float[]{1.2f, 1.5f, 1.5f, 2f, 1.2f, 2f, 1.5f});

            String[] headers = {"No", "Correlativo", "Fecha Pago", "Cliente", "Mesa", "Mesero", "Monto Pagado"};
            for (String h : headers) {
                PdfPCell cell = new PdfPCell(new Phrase(h, FontFactory.getFont(FontFactory.HELVETICA_BOLD, 10)));
                cell.setBackgroundColor(BaseColor.LIGHT_GRAY);
                cell.setHorizontalAlignment(Element.ALIGN_CENTER);
                table.addCell(cell);
            }

            for (IngresoDTO dto : data) {
                table.addCell(dto.getOrdenId().toString());
                table.addCell(dto.getCorrelativo());
                table.addCell(dto.getFechaPago().toString());
                table.addCell(dto.getClienteNombre());
                table.addCell(dto.getMesaNumero().toString());
                table.addCell(dto.getMeseroNombre());
                table.addCell("$ " + dto.getSumaPagos().setScale(2, RoundingMode.HALF_UP).toString());
            }

            document.add(table);

        } else {
            // resumen agrupado por fechaPago
            Map<LocalDate, BigDecimal> resumen = data.stream()
                    .collect(Collectors.groupingBy(
                            IngresoDTO::getFechaPago,
                            Collectors.reducing(BigDecimal.ZERO, IngresoDTO::getSumaPagos, BigDecimal::add)
                    ));

            PdfPTable table = new PdfPTable(2);
            table.setWidthPercentage(50);
            table.setHorizontalAlignment(Element.ALIGN_CENTER);
            table.setWidths(new float[]{2f, 2f});

            String[] headers = {"Fecha", "Total Día"};
            for (String h : headers) {
                PdfPCell cell = new PdfPCell(new Phrase(h, FontFactory.getFont(FontFactory.HELVETICA_BOLD, 11)));
                cell.setBackgroundColor(BaseColor.LIGHT_GRAY);
                cell.setHorizontalAlignment(Element.ALIGN_CENTER);
                table.addCell(cell);
            }

            for (Map.Entry<LocalDate, BigDecimal> entry : resumen.entrySet()) {
                table.addCell(entry.getKey().toString());
                table.addCell("$ " + entry.getValue().setScale(2, RoundingMode.HALF_UP).toString());
            }

            document.add(table);
        }

        // ========= Total general =========
        BigDecimal totalGeneral = data.stream()
                .map(IngresoDTO::getSumaPagos)
                .reduce(BigDecimal.ZERO, BigDecimal::add);

        document.add(new Paragraph(" "));
        Paragraph total = new Paragraph("TOTAL GENERAL: $ " + totalGeneral.setScale(2, RoundingMode.HALF_UP),
                FontFactory.getFont(FontFactory.HELVETICA_BOLD, 12, BaseColor.BLACK));
        total.setAlignment(Element.ALIGN_RIGHT);
        document.add(total);

        document.close();
        return baos.toByteArray();
    }
}

```
## 5. Programar el endpoint en el controlador

```Java
@RestController
@RequestMapping("/api/reportes")
@RequiredArgsConstructor
public class ReporteController {
    @Autowired
    private ReporteService reporteService;

    @GetMapping("/ingresos")
    public ResponseEntity<byte[]> generarReporte(
            @RequestParam("fechaInicio") @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate fechaInicio,
            @RequestParam("fechaFinal") @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate fechaFinal,
            @RequestParam(defaultValue = "true") boolean detallado) throws Exception {

        byte[] pdf = reporteService.generarReporte(fechaInicio, fechaFinal, detallado);

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_PDF);
        headers.setContentDispositionFormData("filename", "reporte_ingresos.pdf");

        return new ResponseEntity<>(pdf, headers, HttpStatus.OK);
    }
}
```



