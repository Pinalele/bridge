import org.springframework.stereotype.Component;
import org.springframework.ws.context.MessageContext;
import org.springframework.ws.server.EndpointInterceptor;
import org.springframework.ws.soap.SoapMessage;
import org.springframework.ws.soap.saaj.SaajSoapMessage;
import org.w3c.dom.Element;
import org.w3c.dom.NodeList;

import javax.xml.soap.SOAPBody;
import javax.xml.soap.SOAPElement;
import javax.xml.soap.SOAPException;

@Component
public class YourEndpointInterceptor implements EndpointInterceptor {

    private static final String NAMESPACE_URI = "http://authentication.dc.opencontent.tsp.com";

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
            NodeList newSessionSimpleResponseList = soapBody.getElementsByTagNameNS(NAMESPACE_URI, "newSessionSimpleResponse");
            if (newSessionSimpleResponseList.getLength() > 0) {
                Element newSessionSimpleResponseElement = (Element) newSessionSimpleResponseList.item(0);
                NodeList returnList = newSessionSimpleResponseElement.getElementsByTagNameNS(NAMESPACE_URI, "return");
                if (returnList.getLength() > 0) {
                    Element returnElement = (Element) returnList.item(0);
                    // Add new elements or attributes to returnElement as needed
                    SOAPElement newElement = soapBody.createElementNS(NAMESPACE_URI, "newElement");
                    newElement.setTextContent("Some value");
                    returnElement.appendChild(newElement);
                }
            }
        }
        return true;
    }

    @Override
    public boolean handleFault(MessageContext messageContext, Object endpoint) throws Exception {
        return true;
    }

    @Override
    public void afterCompletion(MessageContext messageContext, Object endpoint, Exception ex) throws Exception {
        // Do cleanup or logging if needed
    }
