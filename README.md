##Concept-Design-of-Embedded-Edge-based-Predictive-maintenance-system-

Introduction:

Machines like motors can fail suddenly and cause loss in industries.
Our goal is to detect problems before failure happens.
The system collects data like vibration and/or temperature from machines.
The embedded device analyzes the data at the edge itself.
If abnormal condition is found, it gives an early warning.
This reduces downtime and saves maintenance cost.
This project is a software-based STM32 simulation (no hardware used).
We write Embedded C code in STM32CubeIDE and test it in proteus.
The system performs basic embedded functions like sensing the physical value and predict the condition.
A Watchdog Timer is added to reset the system if the program hangs.
Everything is tested in simulation before real hardware use.

#Techniques used:

1️ Embedded C – Programming language used to write the firmware
2️ STM32CubeIDE –
Configure peripherals
Write Embedded C code
Build the project
Generate .hex file
3️ Proteus –
Design STM32 virtual circuit
Load .hex file
Run simulation and test Watchdog

#Steps involved:

Data Acquisition – Collect sensor signals from machine.
Signal Processing – Clean and analyse the data.
Condition Monitoring – Detect abnormal patterns.
Decision & Alert System – Generate warning if fault predicted.
Watchdog Concept – Ensures system reliability by resetting during software failure.

#Pin Configuration using STM32CubeMX:
<img width="751" height="696" alt="image" src="https://github.com/user-attachments/assets/bc367c29-59bc-4a04-aef0-be6749bc0d20" />

GPIO Configuration
PA0 - configured as interrupt input (EXTI) for vibration sensor (SW420) 
PB0 -Manual reset button configured as input with pull-up resistor 
PB1-Status LED configured as output pin
<img width="741" height="550" alt="image" src="https://github.com/user-attachments/assets/e7b5c240-e273-4f67-862d-b63371347d3d" />

#Watchdog Timers
Independent Watchdog (IWDG) enabled for system safety 
Window Watchdog (WWDG) configured to prevent system hang
<img width="1067" height="171" alt="image" src="https://github.com/user-attachments/assets/ab98076d-bf82-4c54-beb7-afd996f565dd" />
Watchdog Timers
Independent Watchdog (IWDG) enabled for system safety 
Window Watchdog (WWDG) configured to prevent system hang

Interrupt Configuration
EXTI0 interrupt enabled for PA0 
Used to count vibration pulses in real-time
Clock configuration:
<img width="1787" height="829" alt="image" src="https://github.com/user-attachments/assets/74171d16-1e41-4865-b158-89c978fe8749" />

#STM32CubeIDE code:

#include "main.h"
#include <stdio.h>
#include <string.h>

IWDG_HandleTypeDef hiwdg;

UART_HandleTypeDef huart1;

WWDG_HandleTypeDef hwwdg;


volatile uint32_t pulse_count = 0;      // counts vibration pulses
float temperature = 0;                  // temperature from DS18B20
uint32_t baseline_sum = 0;              // sum for baseline calculation
uint32_t baseline_samples = 0;          // number of samples taken for baseline
uint32_t baseline = 0;                  // calculated baseline
uint8_t baseline_ready = 0;             // flag when baseline is ready
char msg[100];                          // message buffer for UART



void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_USART1_UART_Init(void);
static void MX_WWDG_Init(void);
float DS18B20_ReadTemp(void);
int main(void)
{

  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_USART1_UART_Init();
sprintf(msg,"STM32 Initialized. Starting simulation...\r\n");
HAL_UART_Transmit(&huart1, (uint8_t*)msg, strlen(msg), HAL_MAX_DELAY);
	  while (1)
	  {
		    /* ---- Check manual baseline reset button ---- */
		    if(HAL_GPIO_ReadPin(Manual_baseline_reset_button_GPIO_Port, Manual_baseline_reset_button_Pin) == GPIO_PIN_RESET)
		    {
		        baseline = 0;
		        baseline_sum = 0;
		        baseline_samples = 0;
		        baseline_ready = 0;

	
  }
}

/*
  * @brief System Clock Configuration
  */
void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

  /** Initializes the RCC Oscillators according to the specified parameters
  * in the RCC_OscInitTypeDef structure.
  */
  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_LSI|RCC_OSCILLATORTYPE_HSE;
  RCC_OscInitStruct.HSEState = RCC_HSE_ON;
  RCC_OscInitStruct.HSEPredivValue = RCC_HSE_PREDIV_DIV1;
  RCC_OscInitStruct.HSIState = RCC_HSI_ON;
  RCC_OscInitStruct.LSIState = RCC_LSI_ON;
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSE;
  RCC_OscInitStruct.PLL.PLLMUL = RCC_PLL_MUL9;
  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    Error_Handler();
  }

  /** Initializes the CPU, AHB and APB buses clocks
  */
  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK|RCC_CLOCKTYPE_SYSCLK
                              |RCC_CLOCKTYPE_PCLK1|RCC_CLOCKTYPE_PCLK2;
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV2;
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;

  if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_2) != HAL_OK)
  {
    Error_Handler();
  }
}

/**
  * @brief IWDG Initialization Function
  * @param None
  * @retval None
  */
static void MX_IWDG_Init(void)
{


  hiwdg.Instance = IWDG;
  hiwdg.Init.Prescaler = IWDG_PRESCALER_4;
  hiwdg.Init.Reload = 4095;
  if (HAL_IWDG_Init(&hiwdg) != HAL_OK)
  {
    Error_Handler();
  }

}

/*
  * @brief USART1 Initialization Function
  */
static void MX_USART1_UART_Init(void)
{

  /* USER CODE BEGIN USART1_Init 0 */

  /* USER CODE END USART1_Init 0 */

  /* USER CODE BEGIN USART1_Init 1 */

  /* USER CODE END USART1_Init 1 */
  huart1.Instance = USART1;
  huart1.Init.BaudRate = 9600;
  huart1.Init.WordLength = UART_WORDLENGTH_8B;
  huart1.Init.StopBits = UART_STOPBITS_1;
  huart1.Init.Parity = UART_PARITY_NONE;
  huart1.Init.Mode = UART_MODE_TX_RX;
  huart1.Init.HwFlowCtl = UART_HWCONTROL_NONE;
  huart1.Init.OverSampling = UART_OVERSAMPLING_16;
  if (HAL_UART_Init(&huart1) != HAL_OK)
  {
    Error_Handler();
  }
  /* USER CODE BEGIN USART1_Init 2 */

  /* USER CODE END USART1_Init 2 */

}

/**
  * @brief WWDG Initialization Function
  */
static void MX_WWDG_Init(void)
{

  /* USER CODE BEGIN WWDG_Init 0 */

  /* USER CODE END WWDG_Init 0 */

  /* USER CODE BEGIN WWDG_Init 1 */

  /* USER CODE END WWDG_Init 1 */
  hwwdg.Instance = WWDG;
  hwwdg.Init.Prescaler = WWDG_PRESCALER_1;
  hwwdg.Init.Window = 64;
  hwwdg.Init.Counter = 64;
  hwwdg.Init.EWIMode = WWDG_EWI_DISABLE;
  if (HAL_WWDG_Init(&hwwdg) != HAL_OK)
  {
    Error_Handler();
  }
  /* USER CODE BEGIN WWDG_Init 2 */

  /* USER CODE END WWDG_Init 2 */

}

/**
  * @brief GPIO Initialization Function
  * @param None
  * @retval None
  */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};
  /* USER CODE BEGIN MX_GPIO_Init_1 */

  /* USER CODE END MX_GPIO_Init_1 */

  /* GPIO Ports Clock Enable */
  __HAL_RCC_GPIOC_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();
  __HAL_RCC_GPIOA_CLK_ENABLE();
  __HAL_RCC_GPIOB_CLK_ENABLE();

  /*Configure GPIO pin Output Level */
  HAL_GPIO_WritePin(Status_LED_GPIO_Port, Status_LED_Pin, GPIO_PIN_RESET);

  /*Configure GPIO pin Output Level */
  HAL_GPIO_WritePin(Manual_baseline_reset_button_GPIO_Port, Manual_baseline_reset_button_Pin, GPIO_PIN_RESET);

  /*Configure GPIO pin : Status_LED_Pin */
  GPIO_InitStruct.Pin = Status_LED_Pin;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(Status_LED_GPIO_Port, &GPIO_InitStruct);

  /*Configure GPIO pin : PA0 */
  GPIO_InitStruct.Pin = GPIO_PIN_0;
  GPIO_InitStruct.Mode = GPIO_MODE_IT_RISING;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

  /*Configure GPIO pin : Manual_baseline_reset_button_Pin */
  GPIO_InitStruct.Pin = Manual_baseline_reset_button_Pin;
  GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
  GPIO_InitStruct.Pull = GPIO_PULLUP;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(Manual_baseline_reset_button_GPIO_Port, &GPIO_InitStruct);

  /* EXTI interrupt init*/
  HAL_NVIC_SetPriority(EXTI0_IRQn, 0, 0);
  HAL_NVIC_EnableIRQ(EXTI0_IRQn);

  /* USER CODE BEGIN MX_GPIO_Init_2 */

  /* USER CODE END MX_GPIO_Init_2 */
}

/* USER CODE BEGIN 4 */

/* USER CODE BEGIN 4 */





float DS18B20_ReadTemp(void)
{
    return 28.0;
}

void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    if(GPIO_Pin == GPIO_PIN_0)
    {
        pulse_count++;
    }
}


/* USER CODE END 4 */
/* Interrupt callback for vibration sensor */

/* USER CODE END 4 */

/**
  * @brief  This function is executed in case of error occurrence.
  * @retval None
  */
void Error_Handler(void)
{
  /* USER CODE BEGIN Error_Handler_Debug */
  /* User can add his own implementation to report the HAL error return state */
  __disable_irq();
  while (1)
  {


}
#ifdef USE_FULL_ASSERT
/**
  * @brief  Reports the name of the source file and the source line number
  *         where the assert_param error has occurred.
  * @param  file: pointer to the source file name
  * @param  line: assert_param error line source number
  * @retval None
  */
void assert_failed(uint8_t *file, uint32_t line)
{
  /* USER CODE BEGIN 6 */
  /* User can add his own implementation to report the file name and line number,
     ex: printf("Wrong parameters value: file %s on line %d\r\n", file, line) */
  /* USER CODE END 6 */
}
#endif /* USE_FULL_ASSERT */
}
#Proteus Equivalent:
<img width="866" height="768" alt="image" src="https://github.com/user-attachments/assets/bd756823-a6ba-469d-87ca-20fc8d06dddb" />

#Prediction:

Five reading have been taken for testing.
Our baseline vibration value was fixed at 8 and normal temperature below 40.
 When both values are under threshold the status is good and LED is off as shown in reading 1.
If anyone is more, it predicts the abnormality and LED will get turned on and status as alert sent to the user.
If both are abnormal or above threshold, then then fault alert is sent to the user.


#Advantages:

Early Fault Detection
Low Cost Implementation
Simple and Efficient Design


#Conclusions:

Summary of Work 
Developed a machine health monitoring system using vibration and temperature inputs 
Implemented pulse counting, baseline learning, and condition classification using STM32 

Key Learning Outcomes 
Understanding of interrupt-based pulse counting 
Sensor interfacing and real-time monitoring 
Embedded system design and decision-making logic.








