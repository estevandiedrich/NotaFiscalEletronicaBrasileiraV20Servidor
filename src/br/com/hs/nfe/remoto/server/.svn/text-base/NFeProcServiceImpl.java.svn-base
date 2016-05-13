package br.com.hs.nfe.remoto.server;


import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.net.URL;
import java.util.List;
import java.util.zip.Adler32;
import java.util.zip.CheckedOutputStream;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;

import javax.activation.DataHandler;
import javax.activation.DataSource;
import javax.activation.FileDataSource;
import javax.jws.WebParam;
import javax.jws.WebService;

import org.apache.log4j.Logger;
import org.apache.log4j.PropertyConfigurator;

import br.com.hs.nfe.conf.Config;
import br.com.hs.nfe.dao.NFeDAO;
import br.com.hs.nfe.entity.Nfe;
import br.com.hs.nfe.hb.NFeUpdateCommand;
import br.com.hs.nfe.vo.ConfigVO;

@WebService(endpointInterface="br.com.hs.nfe.remoto.server.NFeProcService",
	serviceName="NFeProcService")
public class NFeProcServiceImpl implements NFeProcService{
	public NFeProcServiceImpl()
	{
		System.setProperty("nfe.hibernate.conf", "C:\\dev.estevan\\workspace-java\\.metadata\\.plugins\\org.eclipse.wst.server.core\\tmp0\\wtpwebapps\\NotaFiscalEletronicaBrasileiraV20Servidor\\WEB-INF\\classes\\hibernate.cfg.xml");
		System.setProperty("nfe.config.xml","C:\\dev.estevan\\workspace-java\\.metadata\\.plugins\\org.eclipse.wst.server.core\\tmp0\\wtpwebapps\\NotaFiscalEletronicaBrasileiraV20Servidor\\WEB-INF\\classes\\conf\\config.xml");
		URL log4jConfigurationFile = Thread.currentThread().getContextClassLoader().getResource("log4j.properties");
		PropertyConfigurator.configure(log4jConfigurationFile);
		logger.info("Classe construida");
	}
	private static final Logger logger = Logger.getLogger("NFeProcServiceImpl");
	private static final int BUFFER = 2048;
	private NFeDAO nfeDao = new NFeDAO();
	
	public DataHandler getNFeProc(@WebParam(name="cnpj") String cnpj)
	{
		logger.info("Metodo getNFeProc chamado");
		List<Nfe> nfeList = nfeDao.procuraNotasProcessadas(cnpj);
		ConfigVO config = Config.getInstance().getConfig(cnpj);
		DataHandler dataHandler = null;
		BufferedInputStream origin = null;
		FileOutputStream dest = null;
		try 
		{
			logger.info("Criando arquivo zip");
			dest = new FileOutputStream("c:/temp1/"+cnpj+".zip");
			CheckedOutputStream checksum = new CheckedOutputStream(dest, new Adler32());
			ZipOutputStream out = new ZipOutputStream(new BufferedOutputStream(checksum));
			//out.setMethod(ZipOutputStream.DEFLATED);
			byte data[] = new byte[BUFFER];
			for(Nfe nfe:nfeList)
			{
				logger.info("Adding: "+config.getEnviNFeXMLProcessados()+File.separatorChar+nfe.getChaveAcesso()+"-nfeProc.xml");
	            FileInputStream fi = new FileInputStream(config.getEnviNFeXMLProcessados()+File.separatorChar+nfe.getChaveAcesso()+"-nfeProc.xml");
	            origin = new BufferedInputStream(fi, BUFFER);
	            ZipEntry entry = new ZipEntry(config.getEnviNFeXMLProcessados()+File.separatorChar+nfe.getChaveAcesso()+"-nfeProc.xml");
	            out.putNextEntry(entry);
	            int count;
	            while((count = origin.read(data, 0, BUFFER)) != -1) 
	            {
	               out.write(data, 0, count);
	            }
	            origin.close();
	            
	            nfe.setDanfeGerado('1');
				NFeUpdateCommand updateCommand = new NFeUpdateCommand(nfe);
				updateCommand.execute();
			}
			out.close();
			//TODO: Zipar todas as notas e enviar para o cliente gerar o danfe e imprimir
			logger.info("checksum: "+checksum.getChecksum().getValue());
		} 
		catch (FileNotFoundException e) 
		{
			logger.error(e);
		} 
		catch(IOException ioe)
		{
			logger.error(ioe);
		}
		DataSource source = new FileDataSource(new File("c:/temp1/"+cnpj+".zip"));
		dataHandler = new DataHandler(source);
		return dataHandler;
	}
}
