## varints
msb : most significant bit 0/1 1说明后面的字节还是属于当前数据的,如果是0,那么这是当前数据的最后一个字节数据    
varint中的每个字节,除了最后一个字节(实际看到的是反转后的,故每个字节的首位都是msb),都设置了最高有效位(msb),后七位是数据,换算时,首先将每个字节的msb剔除,然后将每个字节反转