import org.springframework.stereotype.Component;
import org.springframework.ws.context.MessageContext;
import org.springframework.ws.server.EndpointInterceptor;
import org.springframework.ws.soap.SoapMessage;
import org.springframework.ws.soap.saaj.SaajSoapMessage;
import javax.xml.soap.SOAPBody;
import javax.xml.soap.SOAPElement;
import javax.xml.soap.SOAPException;

@Component
public class YourEndpointInterceptor implements EndpointInterceptor {

    private static final String NAMESPACE_URI = "http://authentication.dctm.opencontent.tsgrp.com/xsd";

    @Override
    public boolean handleRequest(MessageContext messageContext, Object endpoint) throws Exception {
        return true;
    }

    @Override
    public boolean handleResponse(MessageContext messageContext, Object endpoint) throws Exception {
        SoapMessage soapMessage = (SoapMessage) messageContext.getResponse();
        if (soapMessage instanceof SaajSoapMessage) {
            SaajSoapMessage saajSoapMessage = (SaajSoapMessage) soapMessage;
            SOAPBody soapBody = saajSoapMessage.getSaajMessage().getSOAPBody();
            SOAPElement newSessionSimpleResponseElement = findNewSessionSimpleResponseElement(soapBody);
            if (newSessionSimpleResponseElement != null) {
                // Remove xmlns:ax21 attribute from newSessionSimpleResponseElement
                newSessionSimpleResponseElement.removeAttribute("xmlns:ax21");

                // Find or create the <ns:return> element
                SOAPElement returnElement = findOrCreateReturnElement(soapBody);

                // Add xmlns:ax21 attribute to returnElement
                returnElement.setAttribute("xmlns:ax21", NAMESPACE_URI);
            }
        }
        return true;
    }

    private SOAPElement findNewSessionSimpleResponseElement(SOAPBody soapBody) throws SOAPException {
        // Find the <ns:newSessionSimpleResponse> element
        return (SOAPElement) soapBody.getElementsByTagNameNS(NAMESPACE_URI, "newSessionSimpleResponse").item(0);
    }

    private SOAPElement findOrCreateReturnElement(SOAPBody soapBody) throws SOAPException {
        // Find or create the <ns:return> element
        SOAPElement returnElement = (SOAPElement) soapBody.getElementsByTagNameNS(NAMESPACE_URI, "return").item(0);
        if (returnElement == null) {
            returnElement = soapBody.addChildElement("return", "ns");
            returnElement.setAttribute("xmlns:ns", NAMESPACE_URI);
        }
        return returnElement;
    }

    @Override
    public boolean handleFault(MessageContext messageContext, Object endpoint) throws Exception {
        return true;
    }

    @Override
    public void afterCompletion(MessageContext messageContext, Object endpoint, Exception ex) throws Exception {
        // Do cleanup or logging if needed
    }
}
