Chip lowest posible turn on voltage lay in 2.6-2.8V range. 

On 3.3V one can expect chip to realiably work with 1MBit SPI transfer clock. 

On 5V 2.25MHZ SPI clock was achived without CRC errors.


Main clock frequency 4MHz



stm32 spi peripheral setting is

```c

  hspi1.Instance = SPI1;
  hspi1.Init.Mode = SPI_MODE_MASTER;
  hspi1.Init.Direction = SPI_DIRECTION_1LINE;
  hspi1.Init.DataSize = SPI_DATASIZE_8BIT;
  hspi1.Init.CLKPolarity = SPI_POLARITY_HIGH;//Overclocking might change that
  hspi1.Init.CLKPhase = SPI_PHASE_2EDGE;
  hspi1.Init.NSS = SPI_NSS_SOFT;
  hspi1.Init.BaudRatePrescaler = SPI_BAUDRATEPRESCALER_64;//pay atention here, take main cpu frequency and wich SPI perh is used, for SPI2 on stm32f103 that will result is 2 time slower
  hspi1.Init.FirstBit = SPI_FIRSTBIT_MSB;
  hspi1.Init.TIMode = SPI_TIMODE_DISABLE;
  hspi1.Init.CRCCalculation = SPI_CRCCALCULATION_DISABLE;
  hspi1.Init.CRCPolynomial = 10;
  if (HAL_SPI_Init(&hspi1) != HAL_OK)
  {
    Error_Handler();
  }

```


c code to validate CRC
```c
const uint8_t crcTable[256] = { 0, 0x1d, 0x3a, 0x27, 0x74, 0x69, 0x4e, 0x53, 0xe8,
        0xf5, 0xd2, 0xcf, 0x9c, 0x81, 0xa6, 0xbb, 0xcd, 0xd0, 0xf7, 0xea, 0xb9,
        0xa4, 0x83, 0x9e, 0x25, 0x38, 0x1f, 0x2, 0x51, 0x4c, 0x6b, 0x76, 0x87,
        0x9a, 0xbd, 0xa0, 0xf3, 0xee, 0xc9, 0xd4, 0x6f, 0x72, 0x55, 0x48, 0x1b,
        0x6, 0x21, 0x3c, 0x4a, 0x57, 0x70, 0x6d, 0x3e, 0x23, 0x4, 0x19, 0xa2,
        0xbf, 0x98, 0x85, 0xd6, 0xcb, 0xec, 0xf1, 0x13, 0xe, 0x29, 0x34, 0x67,
        0x7a, 0x5d, 0x40, 0xfb, 0xe6, 0xc1, 0xdc, 0x8f, 0x92, 0xb5, 0xa8, 0xde,
        0xc3, 0xe4, 0xf9, 0xaa, 0xb7, 0x90, 0x8d, 0x36, 0x2b, 0xc, 0x11, 0x42,
        0x5f, 0x78, 0x65, 0x94, 0x89, 0xae, 0xb3, 0xe0, 0xfd, 0xda, 0xc7, 0x7c,
        0x61, 0x46, 0x5b, 0x8, 0x15, 0x32, 0x2f, 0x59, 0x44, 0x63, 0x7e, 0x2d,
        0x30, 0x17, 0xa, 0xb1, 0xac, 0x8b, 0x96, 0xc5, 0xd8, 0xff, 0xe2, 0x26,
        0x3b, 0x1c, 0x1, 0x52, 0x4f, 0x68, 0x75, 0xce, 0xd3, 0xf4, 0xe9, 0xba,
        0xa7, 0x80, 0x9d, 0xeb, 0xf6, 0xd1, 0xcc, 0x9f, 0x82, 0xa5, 0xb8, 0x3,
        0x1e, 0x39, 0x24, 0x77, 0x6a, 0x4d, 0x50, 0xa1, 0xbc, 0x9b, 0x86, 0xd5,
        0xc8, 0xef, 0xf2, 0x49, 0x54, 0x73, 0x6e, 0x3d, 0x20, 0x7, 0x1a, 0x6c,
        0x71, 0x56, 0x4b, 0x18, 0x5, 0x22, 0x3f, 0x84, 0x99, 0xbe, 0xa3, 0xf0,
        0xed, 0xca, 0xd7, 0x35, 0x28, 0xf, 0x12, 0x41, 0x5c, 0x7b, 0x66, 0xdd,
        0xc0, 0xe7, 0xfa, 0xa9, 0xb4, 0x93, 0x8e, 0xf8, 0xe5, 0xc2, 0xdf, 0x8c,
        0x91, 0xb6, 0xab, 0x10, 0xd, 0x2a, 0x37, 0x64, 0x79, 0x5e, 0x43, 0xb2,
        0xaf, 0x88, 0x95, 0xc6, 0xdb, 0xfc, 0xe1, 0x5a, 0x47, 0x60, 0x7d, 0x2e,
        0x33, 0x14, 0x9, 0x7f, 0x62, 0x45, 0x58, 0xb, 0x16, 0x31, 0x2c, 0x97,
        0x8a, 0xad, 0xb0, 0xe3, 0xfe, 0xd9, 0xc4 };


uint8_t TLE5010CRC(uint8_t *buf, uint8_t len) {
    const uint8_t *ptr = buf;
    uint8_t _crc = 0xFB;//0xFF if u want to pass all 6 bytes(zero and command included)

    while (len--)
        _crc = crcTable[_crc ^ *ptr++];

    return ~_crc;
}
```

usage

``` c
SpiSend([0x00, 0x8c]);
uint8_t tle_data[5] = SpiRecive();

crc = TLE5010CRC(tle_data, 4);

if(crc != tle_data[4]) {'crc mismatch'}
```
