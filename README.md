# skill-lab
vlsi lnt lab repository


notes:

SMOKE DETECTING USING STM32
mq-2 sensor for smoke detection - connected to 5v on stm32 board - because the coil must heat up
tin dioxide is the sensitive element present inside - SnO2
clean air- high resistance and smokes and dust reduces the resistance.
4 terminals - 2 analog and 2 digital
AO - provides continuous smoke level information. output voltage varies with smoke concentration 
DO - used for onboard threshold comparator, it gives only high or low
MQ2
vcc - 5v
gnd - gnd 
AO - PAO(adc input)

buzzer  
positive - PA5
negative - gnd

code:

#include "main.h"

/* Peripheral handles */
ADC_HandleTypeDef hadc1;

/* Pin definitions */
#define BUZZER_PIN GPIO_PIN_0
#define BUZZER_PORT GPIOB

/* Smoke threshold */
#define SMOKE_THRESHOLD 1500

uint32_t smoke_value = 0;

/* Function declarations */
void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_ADC1_Init(void);
void Error_Handler(void);

int main(void)
{
    HAL_Init();

    SystemClock_Config();

    MX_GPIO_Init();

    MX_ADC1_Init();

    /* Initially buzzer OFF */
    HAL_GPIO_WritePin(BUZZER_PORT,
                      BUZZER_PIN,
                      GPIO_PIN_RESET);

    while (1)
    {
        /* Start ADC conversion */
        HAL_ADC_Start(&hadc1);

        /* Wait for conversion */
        HAL_ADC_PollForConversion(&hadc1, HAL_MAX_DELAY);

        /* Read MQ2 sensor */
        smoke_value = HAL_ADC_GetValue(&hadc1);

        /* Stop ADC */
        HAL_ADC_Stop(&hadc1);

        /* Check smoke level */
        if (smoke_value > SMOKE_THRESHOLD)
        {
            HAL_GPIO_WritePin(BUZZER_PORT,
                              BUZZER_PIN,
                              GPIO_PIN_SET);
        }
        else
        {
            HAL_GPIO_WritePin(BUZZER_PORT,
                              BUZZER_PIN,
                              GPIO_PIN_RESET);
        }

        HAL_Delay(200);
    }
}

/* ADC1 Initialization */
static void MX_ADC1_Init(void)
{
    ADC_ChannelConfTypeDef sConfig = {0};

    hadc1.Instance = ADC1;

    hadc1.Init.ScanConvMode = ADC_SCAN_DISABLE;
    hadc1.Init.ContinuousConvMode = DISABLE;
    hadc1.Init.DiscontinuousConvMode = DISABLE;
    hadc1.Init.ExternalTrigConv = ADC_SOFTWARE_START;
    hadc1.Init.DataAlign = ADC_DATAALIGN_RIGHT;
    hadc1.Init.NbrOfConversion = 1;

    if (HAL_ADC_Init(&hadc1) != HAL_OK)
    {
        Error_Handler();
    }

    sConfig.Channel = ADC_CHANNEL_0;
    sConfig.Rank = ADC_REGULAR_RANK_1;
    sConfig.SamplingTime = ADC_SAMPLETIME_55CYCLES_5;

    if (HAL_ADC_ConfigChannel(&hadc1, &sConfig) != HAL_OK)
    {
        Error_Handler();
    }
}

/* GPIO Initialization */
static void MX_GPIO_Init(void)
{
    GPIO_InitTypeDef GPIO_InitStruct = {0};

    __HAL_RCC_GPIOA_CLK_ENABLE();
    __HAL_RCC_GPIOB_CLK_ENABLE();

    /* Buzzer PB0 */
    HAL_GPIO_WritePin(GPIOB, BUZZER_PIN, GPIO_PIN_RESET);

    GPIO_InitStruct.Pin = BUZZER_PIN;
    GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
    GPIO_InitStruct.Pull = GPIO_NOPULL;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;

    HAL_GPIO_Init(GPIOB, &GPIO_InitStruct);

    /* MQ2 sensor PA0 */
    GPIO_InitStruct.Pin = GPIO_PIN_0;
    GPIO_InitStruct.Mode = GPIO_MODE_ANALOG;
    GPIO_InitStruct.Pull = GPIO_NOPULL;

    HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
}

/* ADC MSP Initialization */

/* System Clock Configuration - 72 MHz */
void SystemClock_Config(void)
{
    RCC_OscInitTypeDef RCC_OscInitStruct = {0};
    RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

    __HAL_RCC_AFIO_CLK_ENABLE();
    __HAL_RCC_PWR_CLK_ENABLE();

    RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSE;
    RCC_OscInitStruct.HSEState = RCC_HSE_ON;
    RCC_OscInitStruct.HSEPredivValue = RCC_HSE_PREDIV_DIV1;
    RCC_OscInitStruct.HSIState = RCC_HSI_ON;

    RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
    RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSE;
    RCC_OscInitStruct.PLL.PLLMUL = RCC_PLL_MUL9;

    if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
    {
        Error_Handler();
    }

    RCC_ClkInitStruct.ClockType =
        RCC_CLOCKTYPE_HCLK |
        RCC_CLOCKTYPE_SYSCLK |
        RCC_CLOCKTYPE_PCLK1 |
        RCC_CLOCKTYPE_PCLK2;

    RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
    RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
    RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV2;
    RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;

    if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct,
                            FLASH_LATENCY_2) != HAL_OK)
    {
        Error_Handler();
    }

    __HAL_RCC_ADC_CONFIG(RCC_ADCPCLK2_DIV6);
}

/* Error Handler */
void Error_Handler(void)
{
    __disable_irq();

    while (1)
    {
        HAL_GPIO_TogglePin(GPIOB, BUZZER_PIN);
        HAL_Delay(500);
    }
}




