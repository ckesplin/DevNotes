* Initial state only loads for the first render.  Use useEffect to change it on later renders.

**PDF Wrapper Component**
```js
import html2canvas from "html2canvas";
import jsPDF, { GState } from "jspdf";
import { LegacyRef } from "react";

interface PdfWrapperProps {
	printRef: LegacyRef<HTMLDivElement>;
	children?: React.ReactNode;
}

const PdfWrapper: React.FC<PdfWrapperProps> = ({ printRef, children }) => {
	return (
		<div ref={printRef} style={{ position: "fixed", opacity: 0, zIndex: -9999 }} id="pdf-container">
			{children}
		</div>
	);
};
export default PdfWrapper;

export enum PdfOutputType {
	TO_FILE,
	NEW_TAB,
}

export function generatePdf(params: {
	element: HTMLDivElement | null,
	outputType: PdfOutputType,
	filename?: string,
	isDraft?: boolean,
	onComplete?: () => void
}) {
	const {element, outputType, filename="PDF", isDraft=false, onComplete} = params;

	if (element) {
		html2canvas(element, {
			scale: 2,
			onclone: (clonedElement) => {
				const pdfContainer = clonedElement.getElementById("pdf-container");
				if (pdfContainer) {
					pdfContainer.style.opacity = "1";
				}
			},
		}).then((canvas) => {
			const imgData = canvas.toDataURL("image/png");
			const pdf = new jsPDF("p", "in", "letter", true);

			const pageHeight = pdf.internal.pageSize.getHeight();
			const imgWidth = pdf.internal.pageSize.getWidth();
			const imgHeight = (canvas.height * imgWidth) / canvas.width;
			let remainingHeight = imgHeight;
			let position = 0;

			pdf.setProperties({ title: filename });

			pdf.addImage(imgData, "PNG", 0, position, imgWidth, imgHeight, "FAST");
			isDraft && addWatermark(pdf);
			remainingHeight -= pageHeight;

			while (remainingHeight >= 0) {
				position = remainingHeight - imgHeight;
				pdf.addPage();
				pdf.addImage(imgData, "PNG", 0, position, imgWidth, imgHeight, "FAST");
				isDraft && addWatermark(pdf);
				remainingHeight -= pageHeight;
			}

			if (outputType === PdfOutputType.TO_FILE) pdf.save(filename);
			if (outputType === PdfOutputType.NEW_TAB) window.open(pdf.output("bloburl"));
		}).finally(() => {
			onComplete && onComplete()
		});
	}
}

function addWatermark(pdf: jsPDF) {
	pdf.setFont("Helvetica");
	pdf.setFontSize(150);
	pdf.setTextColor("#000");
	pdf.saveGraphicsState();
	pdf.setGState(new GState({ opacity: 0.15 }));
	pdf.text("DRAFT", 1.5, 2.5, { lineHeightFactor: 0, angle: 52.3, rotationDirection: 0 });
	pdf.restoreGraphicsState();
}
```