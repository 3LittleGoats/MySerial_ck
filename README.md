# MySerial_ck

The story behind MySerial_ck:
    After making MySerial (see MySerial repo), I realized I could make a 2-wire protocol, a 4-wire, an 8-wire, and maybe even a 16-wire (did not happen). I also noticed that if i went that far, my main bottleneck would be the fact that the transmitter and receiver are kept in sync by sending a resync cycle every byte. With an 8-wire serial protocol, this would mean 1 bit-time for a byte (because it would send 8 bits at a time), then 2 bit-times just to resync. Two thirds of transmission time on average would be used to keep the transmitter and receiver in sync! The math works out to this: 8 times the wires, a little more than 3 times the transmission speed. I figured that if I was really going to get something good out of multiwire data transmission, it would be best to add one wire dedicated to sending a clock pulse (as is the case with I2C and SPI), which would altogether elliminate the need to have a resync period.
    MySerial_ck was just going to be an experiment to make sure I understood how to do synchronization over a separate clock wire. However, it turned out to be much faster than MySerial! It seemed to send reliably at 100kbps, 10 times the speed! ( It actually does seem to have some minor data corruption issues running at that speed, at least with the Arduino, as I found out while finishing up MySerial4_ck and MySerial8_ck. Instead, I would rate it at 56kbps. Still, a lot faster than 10kbps.)


Technical information:
    Class name: MySerial_ck
    
    Functions in this class:
        begin_t(byte pin1, byte pin2, unsigned int bitTime, unsigned int lineUpTime)
            This function is used to set the serial up as a transmitter
            'pin1' is the digital pin used to transmit data
            'pin2' is the digital pin used to send a clock signal
            'bitTime' is the time between each bit in microseconds (uS) (1,000,000/bitrate)
            'lineUpTime' is the time that the transmitter initially pulls the clock pin high for, in order to "arm" the receiver (let it know that there is incoming data). This is optional, and the default value of 100 should work just fine
        
        begin_r(byte pin1, byte pin2)
            This function is used to set the serial up as a receiver
            'pin1' is the digital pin used to receive data
            'pin2' is the digital pin used to receive a clock signal
        
        bool signal()
            This function tells whether or not there is an incoming message ('true' for incoming message, 'false' for no incoming message)
        
        listen()
            This function checks for an incoming message (I may merge 'signal' and 'listen' into one function in the future to make things a little simpler)
        
        int receive(byte _buf[255])
            This function receives incoming data
            '_buf' is where the bytes of data are stored (the array you put in this spot needn't hold 255 bytes. 255 is the absolute maximum number of bytes that can be received at once. If a smaller array is used, use caution, as Atmel chips [at least the ATMEGA] lock up if an array index gets out of range)
            This function returns the number of bytes it received (-1 if it timed out waiting for data)
        
        transmit(byte _buf[255], int byteTotal)
            This function transmits data
            '_buf' is where the bytes of data to be sent are stored (the array you put in this spot needn't hold 255 bytes. 255 is the absolute maximum number of bytes that can be received at once, so no more should be transmitted at once)
            'byteTotal' is the number of bytes to be sent. Telling the transmitter explicitly how many bytes are to be sent allows the use of a large byte array (such as 255) to be used to send varying numbers of bytes without having to change the length of that array to match the number of bytes being sent
    
    
    minimum number of bytes that can be transmitted/received: 0 (-1 has not been tested :) )
    
    maximum number of bytes that can be transmitted/received: 255 (this is because the very first byte of data sent tells the receiver how many bytes to receive)
    
    minimum bitrate: ~1k
    maximum stable bitrate: 56k (at 100k, 1 byte per ~3000 would be corrupted; at 500k 1 byte per 10)
    maximum bit-time (corresponds to minimum bitrate): ~1000
    minimum bit-time (corresponds to maximum bitrate): 16
    
    other notes:
        Unlike MySerial, there shouldn't be data corruption issues when using the internal oscillator because the receiver only receives a bit when the transmitter tells it to, taking the receiver's clock out of the picture. However, it is highly suspected that the maximum stable bitrate will be directly proportional to the oscillator speed. For example, with 16MHz oscillator, maximum bitrate is 56kbps; with 8MHz oscillator maximum bitrate will probably be 28kbps.
        This serial protocol can be used for two-way communication over the single data bus (in this case 1 data wire and 1 clock wire) (in which case a delay, preferably the same value as the bit-time, between receiving and then transmitting a response is recommended; a pull-down [not pull-up] resistor on the data wire is also highly recommended)
        The general scheme of this serial protocol is:
            "arm" receiver (unlike MySerial, this only involves pulling the clock pin high. As soon as the clock pin goes low, the first bit is sent)
            send byte of data containing number of bytes in the incoming message
            send first byte of message
            send second byte of message
            ...
            send last byte of message
            
        1 bit is sent every time the clock pin changes state. The clock does not make a complete cycle (off then on, or on then off) for each bit. It instead makes a half cycle



Feel free to suggest a better name for this protocol!
