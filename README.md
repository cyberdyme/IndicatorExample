# IndicatorExample

   public static IObservable<T> IgnoreBetween<T>(this IObservable<T> source, Func<T, bool> start, Func<T, bool> end)        
   {            
        return Observable.Create<T>(observer =>            
        {                
            var startTaking = true;
            return source.Where(item =>
            {                        
                if (start(item))                        
                {
                    startTaking = false;                        
                }
                if (startTaking) return true;
                if (end(item))                        
                {                            
                    startTaking = true;                            
                    return false;                        
                }
                return false;                    
        })                    
        .Subscribe(observer);            
  });        
}
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                    xmlns:m="clr-namespace:Matching.Model"
                    xmlns:c="clr-namespace:Matching.Converters"
                    xmlns:cal="http://www.caliburnproject.org"
                    xmlns:dxp="http://schemas.devexpress.com/winfx/2008/xaml/printing"
                    xmlns:dxgt="http://schemas.devexpress.com/winfx/2008/xaml/grid/themekeys"
                    xmlns:dxe="http://schemas.devexpress.com/winfx/2008/xaml/editors"
                    xmlns:infrastructure="clr-namespace:eSwapCommon.Infrastructure;assembly=eSwapCommon"
                    xmlns:d="clr-namespace:eSwapModel.Domain;assembly=eSwapModel"
                    xmlns:e="clr-namespace:Matching.Extensions"
                    xmlns:viewModels="clr-namespace:Matching.ViewModels">
    
    <Style x:Key="DefaultHeaderPrintHeaderStyle"
      TargetType="dxe:TextEdit" BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintHeaderStyle}}">
    <Setter Property="TextWrapping" Value="Wrap" />
    <Setter Property="Height" Value="Auto"/>
    <Setter Property="Template">
      <Setter.Value>
        <ControlTemplate TargetType="dxe:TextEdit">
          <dxe:TextEdit Margin="2" EditValue="{Binding Column.Header}" IsPrintingMode="True" TextWrapping="Wrap" Height="Auto"/>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>


    <!--SwapGroupAndLegalName group header-->
    <DataTemplate x:Key="SwapGroupAndLegalNameDataTemplate">
        <StackPanel Orientation="Horizontal">
            <TextBlock x:Name="statusField" 
                       Text="{Binding Path=Value}" VerticalAlignment="Center"/>
        </StackPanel>
    </DataTemplate>

    <!--Status column-->
    <DataTemplate x:Key="StatusColumnDataTemplate">
        <StackPanel Orientation="Horizontal">
            <Ellipse x:Name="StatusIndicator" Fill="{DynamicResource GrayBrush3}" Width="15" Height="15" Margin="5,0,3,0"/>
            <TextBlock Text="{Binding Path=RowData.Row.Status.Value}" VerticalAlignment="Center"/>
        </StackPanel>

        <DataTemplate.Triggers>
            <DataTrigger Binding="{Binding RowData.Row.Status}" Value="{x:Static d:MatchingStatus.Matched}">
                <Setter TargetName="StatusIndicator" Property="Fill" Value="{e:StatusBaseToHex  {x:Static d:MatchingStatus.Matched}}"/>
            </DataTrigger>
            <DataTrigger Binding="{Binding RowData.Row.Status}" Value="{x:Static d:MatchingStatus.Unmatched}">
                <Setter TargetName="StatusIndicator" Property="Fill" Value="{e:StatusBaseToHex {x:Static d:MatchingStatus.Unmatched}}"/>
            </DataTrigger>
            <DataTrigger Binding="{Binding RowData.Row.Status}" Value="{x:Static d:MatchingStatus.Error}">
                <Setter TargetName="StatusIndicator" Property="Fill" Value="{e:StatusBaseToHex {x:Static d:MatchingStatus.Error}}"/>
            </DataTrigger>
        </DataTemplate.Triggers>
    </DataTemplate>

    <!--Booking column-->
    <DataTemplate x:Key="BookedDataTemplate">
        <StackPanel x:Name="BookageContainer" Orientation="Horizontal">
            <Ellipse x:Name="StatusIndicator" Fill="{DynamicResource GrayBrush3}" Width="15" Height="15" Margin="5,0,3,0"/>
            <TextBlock Text="{Binding RowData.Row.LinkageBookingStatus.Value}" VerticalAlignment="Center"/>
        </StackPanel>

        <DataTemplate.Triggers>
            <DataTrigger Binding="{Binding RowData.Row.LinkageBookingStatus}" Value="{x:Static d:BookingStatus.Booked}">
                <Setter TargetName="StatusIndicator" Property="Fill" Value="{e:StatusBaseToHex {x:Static d:BookingStatus.Booked}}"/>
            </DataTrigger>
            <DataTrigger Binding="{Binding RowData.Row.LinkageBookingStatus}" Value="{x:Static d:BookingStatus.PartiallyBooked}">
                <Setter TargetName="StatusIndicator" Property="Fill" Value="{e:StatusBaseToHex {x:Static d:BookingStatus.PartiallyBooked}}"/>
            </DataTrigger>
            <DataTrigger Binding="{Binding RowData.Row.LinkageBookingStatus}" Value="{x:Static d:BookingStatus.Unbooked}">
                <Setter TargetName="StatusIndicator" Property="Fill" Value="{e:StatusBaseToHex {x:Static d:BookingStatus.Unbooked}}"/>
            </DataTrigger>
            <DataTrigger Binding="{Binding RowData.Row.LinkageBookingStatus}" Value="{x:Static d:BookingStatus.Error}">
                <Setter TargetName="StatusIndicator" Property="Fill" Value="{e:StatusBaseToHex {x:Static d:BookingStatus.Error}}"/>
            </DataTrigger>
            <DataTrigger Binding="{Binding RowData.Row.LinkageBookingStatus}" Value="{x:Static d:BookingStatus.Unknown}">
                <Setter TargetName="BookageContainer" Property="Visibility" Value="Collapsed"/>
            </DataTrigger>
        </DataTemplate.Triggers>
    </DataTemplate>
    
  <!--Linkage Details Columns-->
    
  <!-- Type Column -->
  <DataTemplate x:Key="TypeColumnDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding Type}" VerticalAlignment="Center">
              <TextBlock.Style>
                <Style TargetType="TextBlock">
                  <Setter Property="Margin" Value="2"/>
                  <Style.Triggers>
                    <DataTrigger Binding="{Binding Type}" Value="BL">
                      <Setter Property="Margin" Value="10,2,2,2"/>
                    </DataTrigger>
                    <DataTrigger Binding="{Binding Type}" Value="AL">
                      <Setter Property="Margin" Value="10,2,2,2"/>
                    </DataTrigger>
                  </Style.Triggers>
                </Style>
              </TextBlock.Style>
            </TextBlock>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="TypeColumnPrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding Type}" VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!-- Booking Linkage Detail -->
    <DataTemplate x:Key="BookingColumnDataTemplate">
        <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
            <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                    <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                            x:Name="Text"
                            cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                            cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">

                        <StackPanel x:Name="BookageContainer" Orientation="Horizontal">
                            <Ellipse x:Name="StatusIndicator" Fill="{DynamicResource GrayBrush3}" Width="15" Height="15" Margin="5,0,3,0"/>
                            <TextBlock Text="{Binding Booking.Value, FallbackValue='Unknown'}"
                                       ToolTip="{Binding Booking.Value}"
                                       HorizontalAlignment="Right"
                                       VerticalAlignment="Center" Margin="2"/>
                        </StackPanel>
                    </Border>

                    <DataTemplate.Triggers>
                        <DataTrigger Value="True">
                            <DataTrigger.Binding>
                                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                                    <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                                    <Binding/>
                                </MultiBinding>
                            </DataTrigger.Binding>
                            <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
                        </DataTrigger>
                        <DataTrigger Binding="{Binding Booking}" Value="{x:Static d:BookingStatus.Booked}">
                            <Setter TargetName="StatusIndicator" Property="Fill" Value="{e:StatusBaseToHex {x:Static d:BookingStatus.Booked}}"/>
                        </DataTrigger>
                        <DataTrigger Binding="{Binding Booking}" Value="{x:Static d:BookingStatus.PartiallyBooked}">
                            <Setter TargetName="StatusIndicator" Property="Fill" Value="{e:StatusBaseToHex {x:Static d:BookingStatus.PartiallyBooked}}"/>
                        </DataTrigger>
                        <DataTrigger Binding="{Binding Booking}" Value="{x:Static d:BookingStatus.Unbooked}">
                            <Setter TargetName="StatusIndicator" Property="Fill" Value="{e:StatusBaseToHex {x:Static d:BookingStatus.Unbooked}}"/>
                        </DataTrigger>
                        <DataTrigger Binding="{Binding Booking}" Value="{x:Static d:BookingStatus.Error}">
                            <Setter TargetName="StatusIndicator" Property="Fill" Value="{e:StatusBaseToHex {x:Static d:BookingStatus.Error}}"/>
                        </DataTrigger>
                        <DataTrigger Binding="{Binding Booking}" Value="{x:Static d:BookingStatus.Unknown}">
                            <Setter TargetName="BookageContainer" Property="Visibility" Value="Hidden"/>
                        </DataTrigger>                        
                    </DataTemplate.Triggers>
                </DataTemplate>
            </ItemsControl.ItemTemplate>
        </ItemsControl>
    </DataTemplate>
    <Style x:Key="BookingPrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
        <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
        <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
        <Setter Property="DisplayTemplate">
            <Setter.Value>
                <ControlTemplate  TargetType="dxe:TextEdit">
                    <Grid>
                        <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
                            <ItemsControl.ItemTemplate>
                                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                                    <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                                            x:Name="Text">
                                        <TextBlock Text="{Binding Booking.Value}"
                                                   HorizontalAlignment="Right"
                                                   VerticalAlignment="Center" Margin="2"/>
                                    </Border>
                                </DataTemplate>
                            </ItemsControl.ItemTemplate>
                        </ItemsControl>
                    </Grid>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>

    <!-- Underlying -->
    <DataTemplate x:Key="UnderlyingDataTemplate">
        <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
            <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                    <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                            x:Name="Text"
                            cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                            cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
                        <TextBlock Text="{Binding Underlying}"
                                   HorizontalAlignment="Right"
                                   TextWrapping="NoWrap"
                                   ToolTip="{Binding Underlying}" VerticalAlignment="Center" Margin="2"/>
                    </Border>

                    <DataTemplate.Triggers>
                        <DataTrigger Value="True">
                            <DataTrigger.Binding>
                                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                                    <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                                    <Binding/>
                                </MultiBinding>
                            </DataTrigger.Binding>
                            <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
                        </DataTrigger>
                    </DataTemplate.Triggers>
                </DataTemplate>
            </ItemsControl.ItemTemplate>
        </ItemsControl>
    </DataTemplate>
    <Style x:Key="UnderlyingPrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
        <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
        <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
        <Setter Property="DisplayTemplate">
            <Setter.Value>
                <ControlTemplate  TargetType="dxe:TextEdit">
                    <Grid>
                        <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
                            <ItemsControl.ItemTemplate>
                                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                                    <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                                            x:Name="Text">
                                        <TextBlock Text="{Binding Underlying}"
                                                   HorizontalAlignment="Right"
                                                   VerticalAlignment="Center" Margin="2"/>
                                    </Border>
                                </DataTemplate>
                            </ItemsControl.ItemTemplate>
                        </ItemsControl>
                    </Grid>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>

    <!-- Gross Price -->
  <DataTemplate x:Key="GrossPriceColumnDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding GrossPrice, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                       ToolTip="{Binding GrossPrice}"
                       HorizontalAlignment="Right"
                       VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="GrossPricePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding GrossPrice, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!-- LocalGrossPrice -->
  <DataTemplate x:Key="LocalGrossPriceColumnDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding LocalGrossPrice, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding LocalGrossPrice}" VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="LocalGrossPricePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding LocalGrossPrice, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!-- Quantity -->
  <DataTemplate x:Key="QuantityColumnDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding Quantity, StringFormat={x:Static Member=infrastructure:StringFormats.CustomCellLongPrecision}}"
                       HorizontalAlignment="Right"
                       VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="QuantityPrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding Quantity}" VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--SettleCurrency-->
  <DataTemplate x:Key="SettleCurrencyDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding SettleCurrency}" VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="SettleCurrencyPrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding SettleCurrency}" VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--CommissionPercentage-->
  <DataTemplate x:Key="CommissionPercentageDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding CommissionPercentage, StringFormat={x:Static infrastructure:StringFormats.CustomPercentagePrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding CommissionPercentage}"
                       VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="CommissionPercentagePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding CommissionPercentage, 
                                            StringFormat={x:Static infrastructure:StringFormats.CustomPercentagePrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--CommissionAmountPerShare-->
  <DataTemplate x:Key="CommissionAmountPerShareDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding CommissionAmountPerShare, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding CommissionAmountPerShare}"
                       VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="CommissionAmountPerSharePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding CommissionAmountPerShare, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--CommissionAbsolute-->
  <DataTemplate x:Key="CommissionAbsoluteDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
                        <TextBlock Text="{Binding CommissionAbsolute, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding CommissionAbsolute}" VerticalAlignment="Center" Margin="2"/>
        </Border>
        <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="CommissionAbsolutePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding CommissionAbsolute, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--Fx Rate -->
  <DataTemplate x:Key="FxRateDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding FxRate, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding FxRate}" VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="FxRatePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding FxRate, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--Business Unit -->
  <DataTemplate x:Key="BusinessUnitDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding BusinessUnit}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding BusinessUnit}" VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="BusinessUnitPrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding BusinessUnit}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!-- Fund -->
  <DataTemplate x:Key="FundDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding Fund}"
                       HorizontalAlignment="Right"
                       TextWrapping="NoWrap"
                       ToolTip="{Binding Fund}" VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="FundPrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding Fund}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!-- Source System -->
  <DataTemplate x:Key="SourceSystemDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding SourceSystem}"
                       HorizontalAlignment="Right"
                       TextWrapping="NoWrap"
                       ToolTip="{Binding SourceSystem}" VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="SourceSystemPrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding SourceSystem}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--Net Notional -->
  <DataTemplate x:Key="NetNotionalDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding NetNotional, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding NetNotional}" VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="NetNotionalPrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding NetNotional, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--ResearchCommissionPercentage-->
  <DataTemplate x:Key="ResearchCommissionPercentageDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding ResearchCommissionPercentage, StringFormat={x:Static infrastructure:StringFormats.CustomPercentagePrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding ResearchCommissionPercentage}"
                       VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="ResearchCommissionPercentagePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding ResearchCommissionPercentage, 
                                            StringFormat={x:Static infrastructure:StringFormats.CustomPercentagePrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--ResearchCommissionAmountPerShare-->
  <DataTemplate x:Key="ResearchCommissionAmountPerShareDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding ResearchCommissionAmountPerShare, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding ResearchCommissionAmountPerShare}"
                       VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="ResearchCommissionAmountPerSharePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding ResearchCommissionAmountPerShare, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--ResearchCommissionAbsolute-->
  <DataTemplate x:Key="ResearchCommissionAbsoluteDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding ResearchCommissionAbsolute, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding ResearchCommissionAbsolute}" VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="ResearchCommissionAbsolutePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding ResearchCommissionAbsolute, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--ExecutionCommissionPercentage-->
  <DataTemplate x:Key="ExecutionCommissionPercentageDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding ExecutionCommissionPercentage, StringFormat={x:Static infrastructure:StringFormats.CustomPercentagePrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding ExecutionCommissionPercentage}"
                       VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="ExecutionCommissionPercentagePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding ExecutionCommissionPercentage, 
                                            StringFormat={x:Static infrastructure:StringFormats.CustomPercentagePrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--ExecutionCommissionAmountPerShare-->
  <DataTemplate x:Key="ExecutionCommissionAmountPerShareDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding ExecutionCommissionAmountPerShare, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding ExecutionCommissionAmountPerShare}"
                       VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="ExecutionCommissionAmountPerSharePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding ExecutionCommissionAmountPerShare, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--ExecutionCommissionAbsolute-->
  <DataTemplate x:Key="ExecutionCommissionAbsoluteDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding ExecutionCommissionAbsolute, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding ExecutionCommissionAbsolute}" VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="ExecutionCommissionAbsolutePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding ExecutionCommissionAbsolute, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>


  <!--MarketFeesPercentage-->
  <DataTemplate x:Key="MarketFeesPercentageDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding MarketFeesPercentage, StringFormat={x:Static infrastructure:StringFormats.CustomPercentagePrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding MarketFeesPercentage}"
                       VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="MarketFeesPercentagePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding MarketFeesPercentage, 
                                            StringFormat={x:Static infrastructure:StringFormats.CustomPercentagePrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--MarketFeesAmountPerShare-->
  <DataTemplate x:Key="MarketFeesAmountPerShareDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding MarketFeesAmountPerShare, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding MarketFeesAmountPerShare}"
                       VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="MarketFeesAmountPerSharePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding MarketFeesAmountPerShare, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--MarketFeesAbsolute-->
  <DataTemplate x:Key="MarketFeesAbsoluteDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding MarketFeesAbsolute, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding MarketFeesAbsolute}" VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="MarketFeesAbsolutePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding MarketFeesAbsolute, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--MarketTaxesPercentage-->
  <DataTemplate x:Key="MarketTaxesPercentageDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding MarketTaxesPercentage, StringFormat={x:Static infrastructure:StringFormats.CustomPercentagePrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding MarketTaxesPercentage}"
                       VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="MarketTaxesPercentagePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding MarketTaxesPercentage, 
                                            StringFormat={x:Static infrastructure:StringFormats.CustomPercentagePrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--MarketTaxesAmountPerShare-->
  <DataTemplate x:Key="MarketTaxesAmountPerShareDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding MarketTaxesAmountPerShare, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding MarketTaxesAmountPerShare}"
                       VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="MarketTaxesAmountPerSharePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding MarketTaxesAmountPerShare, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--MarketTaxesAbsolute-->
  <DataTemplate x:Key="MarketTaxesAbsoluteDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding MarketTaxesAbsolute, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding MarketTaxesAbsolute}" VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="MarketTaxesAbsolutePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding MarketTaxesAbsolute, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--OtherFeesPercentage-->
  <DataTemplate x:Key="OtherFeesPercentageDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding OtherFeesPercentage, StringFormat={x:Static infrastructure:StringFormats.CustomPercentagePrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding OtherFeesPercentage}"
                       VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="OtherFeesPercentagePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding OtherFeesPercentage, 
                                            StringFormat={x:Static infrastructure:StringFormats.CustomPercentagePrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--OtherFeesAmountPerShare-->
  <DataTemplate x:Key="OtherFeesAmountPerShareDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding OtherFeesAmountPerShare, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding OtherFeesAmountPerShare}"
                       VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="OtherFeesAmountPerSharePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding OtherFeesAmountPerShare, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>

  <!--OtherFeesAbsolute-->
  <DataTemplate x:Key="OtherFeesAbsoluteDataTemplate">
    <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
      <ItemsControl.ItemTemplate>
        <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
          <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                  x:Name="Text"
                  cal:Message.Attach="[Event MouseLeftButtonDown] = [Action SetSelectedDetail($datacontext)]"
                  cal:Action.TargetWithoutContext="{Binding ElementName=Root, Path=DataContext}">
            <TextBlock Text="{Binding OtherFeesAbsolute, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                       HorizontalAlignment="Right"
                       ToolTip="{Binding OtherFeesAbsolute}" VerticalAlignment="Center" Margin="2"/>
          </Border>

          <DataTemplate.Triggers>
            <DataTrigger Value="True">
              <DataTrigger.Binding>
                <MultiBinding Converter="{x:Static c:SingleDistinctItemConverter.Instance}">
                  <Binding Path="DataContext.SelectedDetail" ElementName="Root"/>
                  <Binding/>
                </MultiBinding>
              </DataTrigger.Binding>
              <Setter TargetName="Text" Property="TextElement.FontWeight" Value="Bold"/>
            </DataTrigger>
          </DataTemplate.Triggers>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </DataTemplate>
  <Style x:Key="OtherFeesAbsolutePrintStyle" TargetType="dxe:TextEdit"  BasedOn="{StaticResource {dxgt:TableViewThemeKey ResourceKey=DefaultPrintCellStyle}}">
    <Setter Property="dxp:ExportSettings.TargetType" Value="Image" />
    <Setter Property="dxp:ExportSettings.PropertiesHintMask" Value="TargetType" />
    <Setter Property="DisplayTemplate">
      <Setter.Value>
        <ControlTemplate  TargetType="dxe:TextEdit">
          <Grid>
            <ItemsControl ItemsSource="{Binding RowData.Row.DisplayDetails}">
              <ItemsControl.ItemTemplate>
                <DataTemplate DataType="{x:Type viewModels:ILinkageDetail}">
                  <Border Style="{Binding Type, Converter={x:Static c:SubTypeToTextBlockStyleConverter.Instance}}"
                          x:Name="Text">
                    <TextBlock Text="{Binding OtherFeesAbsolute, StringFormat={x:Static infrastructure:StringFormats.CustomCellDecimalPrecision}}"
                               HorizontalAlignment="Right"
                               VerticalAlignment="Center" Margin="2"/>
                  </Border>
                </DataTemplate>
              </ItemsControl.ItemTemplate>
            </ItemsControl>
          </Grid>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>
</ResourceDictionary>

===================================================================================


<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                    xmlns:vm="clr-namespace:Matching.ViewModels"
                    xmlns:cal="http://www.caliburnproject.org"
                    xmlns:dxg="http://schemas.devexpress.com/winfx/2008/xaml/grid"
                    xmlns:dxb="http://schemas.devexpress.com/winfx/2008/xaml/bars"
                    xmlns:dx="http://schemas.devexpress.com/winfx/2008/xaml/core"
                    xmlns:i="http://schemas.microsoft.com/expression/2010/interactivity"
                    xmlns:ip="http://metro.mahapps.com/winfx/xaml/iconpacks"
                    xmlns:dxe="http://schemas.devexpress.com/winfx/2008/xaml/editors"
                    xmlns:behaviours1="clr-namespace:eSwapCommon.Behaviours;assembly=eSwapCommon"
                    xmlns:inf="clr-namespace:eSwapCommon.Infrastructure;assembly=eSwapCommon"
                    xmlns:b="clr-namespace:Matching.Behaviours"
                    xmlns:c="http://metro.mahapps.com/winfx/xaml/controls"
                    xmlns:m="clr-namespace:Matching.Model"
                    xmlns:dxgt="http://schemas.devexpress.com/winfx/2008/xaml/grid/themekeys"
                    xmlns:converters="clr-namespace:eSwapCommon.Converters;assembly=eSwapCommon"
                    xmlns:d="clr-namespace:eSwapModel.Domain;assembly=eSwapModel"
                    xmlns:e="clr-namespace:Matching.Extensions"
                    xmlns:themes="http://schemas.devexpress.com/winfx/2008/xaml/core/themekeys">

    <Style x:Key="{themes:DefaultStyleThemeKey FullName=DevExpress.Xpf.Grid.ColumnChooserControl}" 
           TargetType="{x:Type dxg:ColumnChooserControl}">
    </Style>
    
    
    <DataTemplate x:Key="{dxgt:GroupRowThemeKey ResourceKey=GroupRowTemplate, IsThemeIndependent=True}">
        <dx:MeasurePixelSnapper>
            <dxg:GroupGridRowContent x:Name="PART_GroupRowContent"  Style="{Binding Path=View.GroupRowStyle}" 
                                     Background="{DynamicResource WindowBackgroundBrush}"
                                     Foreground="{DynamicResource BlackBrush}">
                <dx:DXDockPanel>
                    <dxg:GridGroupExpandButton x:Name="Toggle" Margin="5,2,0,3" VerticalAlignment="Center" 
                                            IsChecked="{Binding Path=IsRowExpanded}" Command="{Binding View.Commands.ChangeGroupExpanded}" CommandParameter="{Binding RowHandle.Value}" />
                    <dxg:GroupValueContentPresenter Margin="0,2,0,3" Content="{Binding Path=GroupValue}"
                                                ContentTemplateSelector="{Binding Path=Content.Column.ActualGroupValueTemplateSelector, RelativeSource={RelativeSource Self}}"  />
                    <dxg:GroupSummaryContainer dxg:RowData.CurrentRowData="{Binding}" Name="PART_GroupSummaryPlaceHolder" />
                </dx:DXDockPanel>
            </dxg:GroupGridRowContent>
        </dx:MeasurePixelSnapper>
    </DataTemplate>

    <DataTemplate DataType="{x:Type vm:MatchingViewModel}">
        <Grid x:Name="Root">
            <Grid.RowDefinitions>
                <RowDefinition Height="3*"/>
                <RowDefinition Height="Auto"/>
                <RowDefinition>
                    <RowDefinition.Style>
                        <Style TargetType="RowDefinition">
                            <Setter Property="Height" Value="1*"/>
                            <Style.Triggers>
                                <DataTrigger Binding="{Binding SelectedDetail}" Value="{x:Null}">
                                    <Setter Property="Height" Value="Auto"/>
                                </DataTrigger>
                            </Style.Triggers>
                        </Style>
                    </RowDefinition.Style>
                </RowDefinition>
            </Grid.RowDefinitions>

            <dxg:GridControl Grid.Row="0" Name="GridControl1"
                             AllowLiveDataShaping="True" 
                             AutoGenerateColumns="None" 
                             ItemsSource="{Binding Items}" 
                             SelectionMode="Row"        
                             SelectedItem="{Binding SelectedLinkage}" 
                             SelectedItems="{Binding SelectionItems}"
                             dx:ThemeManager.ThemeName="Office2016WhiteSE" >
                <i:Interaction.Behaviors>
                    <b:RowFilteringBehaviour/>
                    <b:DropFilterBehaviour/>
                    <b:GridControlFilterBehaviour Filter="{Binding Filter}"/>
                </i:Interaction.Behaviors>
                <dxg:GridControl.Resources>
                    <Style TargetType="{x:Type dxg:RowControl}" BasedOn="{StaticResource {x:Type dxg:RowControl}}">
                        <Style.Triggers>
                            <DataTrigger Binding="{Binding Path=Row.TradeViewMode}" Value="{x:Static m:TradeViewModeType.PreviewMode}">
                                <Setter Property="Background" Value="{DynamicResource AccentBaseColorBrush}" />
                                <Setter Property="TextBlock.Foreground" Value="{DynamicResource BlackBrush}" />
                                <Setter Property="FontStyle" Value="Italic" />
                                <Setter Property="FontWeight" Value="DemiBold" />
                            </DataTrigger>
                        </Style.Triggers>
                    </Style>

                    <!-- Show the row indicator if trade is in preview mode sets that  -->
                    <DataTemplate x:Key="{dxgt:RowIndicatorThemeKey ResourceKey=IconPresenterTemplate,IsThemeIndependent=True}">
                        <!-- Set to a minimum height to control row height the indicator field is always shown -->
                        <Grid x:Name="root" Background="{DynamicResource WindowBackgroundBrush}" MinHeight="42">
                            <Rectangle x:Name="matchIndicator" 
                                     Visibility="Collapsed"
                                     Margin="0,3"
                                     Fill="{DynamicResource GrayBrush3}" Width="2" VerticalAlignment="Stretch" HorizontalAlignment="Right"/>

                            <ip:PackIconMaterial x:Name="StatusIndicator"
                                                        Visibility="Collapsed"
                                                        Kind="ChevronDoubleRight" VerticalAlignment="Center" 
                                                        VerticalContentAlignment="Center"
                                                        HorizontalAlignment="Center" Foreground="{DynamicResource GrayBrush1}" 
                                                        Width="10" Height="10"/>
                            <ContentPresenter x:Name="iconPresenter" Content="{x:Null}" />
                        </Grid>
                        <DataTemplate.Triggers>
                            <DataTrigger Binding="{Binding Path=., Converter={x:Static converters:IsAutoFilterRowConverter.Instance}}" Value="True">
                                <Setter TargetName="root" Property="MinHeight" Value="24" />
                            </DataTrigger>
                            <DataTrigger Binding="{Binding Path=Row.TradeViewMode}" Value="{x:Static m:TradeViewModeType.ReadOnly}">
                                <Setter TargetName="matchIndicator" Property="Visibility" Value="Visible" />
                            </DataTrigger>
                            <DataTrigger Binding="{Binding Path=Row.TradeViewMode}" Value="{x:Static m:TradeViewModeType.PreviewMode}">
                                <Setter TargetName="StatusIndicator" Property="Visibility" Value="Visible" />
                            </DataTrigger>
                            <DataTrigger Binding="{Binding Row.Status}" Value="Matched">
                                <Setter TargetName="matchIndicator" Property="Fill" Value="{DynamicResource Brushes.Good}" />
                            </DataTrigger>
                            <DataTrigger Binding="{Binding Row.Status}" Value="Unmatched">
                                <Setter TargetName="matchIndicator" Property="Fill" Value="{DynamicResource Brushes.Warning}" />
                            </DataTrigger>
                        </DataTemplate.Triggers>
                    </DataTemplate>

                    <Style TargetType="dxg:GridColumn">
                        <Setter Property="HeaderTemplate">
                            <Setter.Value>
                                <DataTemplate >
                                    <TextBlock Text="{Binding}" TextWrapping="Wrap"/>
                                </DataTemplate>
                            </Setter.Value>
                        </Setter>
                    </Style>
                </dxg:GridControl.Resources>

                <dxg:GridControl.Bands>
                    <dxg:GridControlBand Name="NormalColumns" VisibleIndex="0" OverlayHeaderByChildren="True">
                        <dxg:GridColumn x:Name="LinkageId" Header="Linkage ID" FieldName="LinkageId" HorizontalHeaderContentAlignment="Center" Visible="False" VisibleIndex="1" />
                        <dxg:GridColumn x:Name="SwapGroup" Header="Swap Group" FieldName="SwapGroup" HorizontalHeaderContentAlignment="Center" Visible="False" VisibleIndex="2"/>
                        <dxg:GridColumn x:Name="SwapGroupLegalName" Header="Legal Name" FieldName="SwapGroupLegalName" HorizontalHeaderContentAlignment="Center" Visible="False" VisibleIndex="3"/>
                        <dxg:GridColumn x:Name="SwapGroupAndLegalName" Header="Swap Group Legal Name" GroupIndex="0" FieldName="SwapGroupAndLegalName" 
                                        GroupValueTemplate="{StaticResource SwapGroupAndLegalNameDataTemplate}" HorizontalHeaderContentAlignment="Center" Width="160" VisibleIndex="4"/>
                        <dxg:GridColumn x:Name="Underlying" FieldName="Underlying" HorizontalHeaderContentAlignment="Center" VisibleIndex="5"/>
                        <dxg:GridColumn x:Name="Side" FieldName="Side" HorizontalHeaderContentAlignment="Center" Width="48" VisibleIndex="6"/>
                        <dxg:GridColumn x:Name="Broker" FieldName="Broker" HorizontalHeaderContentAlignment="Center" VisibleIndex="7"/>
                        <dxg:GridColumn x:Name="TradeDate" Header="Trade Date" FieldName="TradeDate" HorizontalHeaderContentAlignment="Center" Width="80" VisibleIndex="8"/>
                        <dxg:GridColumn x:Name="Client" Header="Client" FieldName="Client" HorizontalHeaderContentAlignment="Center" VisibleIndex="9"/>
                        <dxg:GridColumn x:Name="Notional" Header="Notional" FieldName="NetNotional" HorizontalHeaderContentAlignment="Center" Visible="False" VisibleIndex="10"/>
                        <dxg:GridColumn x:Name="DealStrategy" Header="Deal Strategy" FieldName="DealStrategy" HorizontalHeaderContentAlignment="Center" Visible="False" VisibleIndex="11"/>
                        <dxg:GridColumn x:Name="TradeViewMode" Header="Trade ViewMode" FieldName="TradeViewMode" HorizontalHeaderContentAlignment="Center" Visible="False" VisibleIndex="120"/>
                        <dxg:GridColumn x:Name="AllocationCount" Header="Allocation Count" FieldName="AllocationCount" HorizontalHeaderContentAlignment="Center" Visible="False" VisibleIndex="130"/>
                        <dxg:GridColumn x:Name="BlockCount" Header="Block Count" FieldName="BlockCount" HorizontalHeaderContentAlignment="Center" Visible="False" VisibleIndex="131"/>
                    </dxg:GridControlBand>
                    <dxg:GridControlBand Name="StatusDetails" VisibleIndex="1" Header="Status" >
                        <dxg:GridColumn x:Name="Matched" 
                                        FieldName="{e:EnumToString {x:Static d:DataKeys.Status}}" 
                                        Header="{e:EnumDescriptionAttribute {x:Static d:DataKeys.Status}}" 
                                        HorizontalHeaderContentAlignment="Center" Width="112"
                                        CellTemplate="{StaticResource StatusColumnDataTemplate}" VisibleIndex="130" />

                        <dxg:GridColumn x:Name="Booked" 
                                        FieldName="{e:EnumToString {x:Static d:DataKeys.LinkageBookingStatus}}" 
                                        Header="{e:EnumDescriptionAttribute {x:Static d:DataKeys.LinkageBookingStatus}}"
                                        HorizontalHeaderContentAlignment="Center" Width="96"
                                        CellTemplate="{StaticResource BookedDataTemplate}" VisibleIndex="132"/>
                    </dxg:GridControlBand>

                    <dxg:GridControlBand Name="NormalColumns1" VisibleIndex="2" OverlayHeaderByChildren="True">
                        <dxg:GridColumn Width="32" x:Name="Linkageinfofield" FieldName="RowId" Header=" " AllowAutoFilter="False" AllowColumnFiltering="False" VisibleIndex="140">
                            <dxg:GridColumn.CellTemplate>
                                <DataTemplate>
                                    <Button x:Name="ExpandCollapseButton" 
                                            Foreground="{DynamicResource AccentColorBrush}"
                                            cal:Message.Attach="[Action ToggleSummaryDetails()]"
                                            cal:Action.TargetWithoutContext="{Binding RowData.Row}">
                                        <Button.Style>
                                            <Style TargetType="Button" BasedOn="{StaticResource ChromelessButtonStyle}">
                                                <Setter Property="ToolTip" Value="Show Block and Allocation details."/>
                                                <Style.Triggers>
                                                    <DataTrigger Binding="{Binding RowData.Row.IsDetailed}" Value="True">
                                                        <Setter Property="ToolTip" Value="Hide Block and Allocation details."/>
                                                    </DataTrigger>
                                                </Style.Triggers>
                                            </Style>
                                        </Button.Style>

                                        <ip:PackIconFontAwesome VerticalAlignment="Center" HorizontalAlignment="Center" Width="15" Height="15">
                                            <ip:PackIconFontAwesome.Style>
                                                <Style TargetType="{x:Type ip:PackIconFontAwesome}">
                                                    <Setter Property="ip:PackIconFontAwesome.Kind" Value="Expand"/>
                                                    <Style.Triggers>
                                                        <DataTrigger Binding="{Binding RowData.Row.IsDetailed}" Value="True">
                                                            <Setter Property="ip:PackIconFontAwesome.Kind" Value="Compress"/>
                                                        </DataTrigger>
                                                    </Style.Triggers>
                                                </Style>
                                            </ip:PackIconFontAwesome.Style>
                                        </ip:PackIconFontAwesome>
                                    </Button>
                                    <DataTemplate.Triggers>
                                        <DataTrigger Binding="{Binding RowData.Row.TradeViewMode}" Value="{x:Static m:TradeViewModeType.PreviewMode}">
                                            <Setter TargetName="ExpandCollapseButton" Property="Foreground" Value="{DynamicResource HighlightBrush}" />
                                        </DataTrigger>
                                    </DataTemplate.Triggers>
                                </DataTemplate>
                            </dxg:GridColumn.CellTemplate>
                        </dxg:GridColumn>
                    </dxg:GridControlBand>

                    <dxg:GridControlBand Name="LinkageDetails" Header="Linkage Details" VisibleIndex="3">
                        <dxg:GridColumn Header="Linkage Details" Width="350" HorizontalHeaderContentAlignment="Center" AllowPrinting="False" 
                                        ShowCriteriaInAutoFilterRow="False" AllowAutoFilter="False" AllowColumnFiltering="False" Visible="False">
                            <dxg:GridColumn.FilterEditorHeaderTemplate>
                                <DataTemplate/>
                            </dxg:GridColumn.FilterEditorHeaderTemplate>
                            <dxg:GridColumn.CellTemplate>
                                <DataTemplate/>
                            </dxg:GridColumn.CellTemplate>
                        </dxg:GridColumn>
                        <dxg:GridColumn FieldName="Type" VisibleIndex="10" GridRow="0" HorizontalHeaderContentAlignment="Center" UnboundType="String"
                                        Width="40" CellTemplate="{StaticResource TypeColumnDataTemplate}" PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" PrintCellStyle="{StaticResource TypeColumnPrintStyle}" />

                        <dxg:GridColumn GridRow="0" HorizontalHeaderContentAlignment="Center" UnboundType="Object"
                                        FieldName="{e:EnumToString {x:Static d:DataKeys.Booking}}" 
                                        Header="{e:EnumDescriptionAttribute {x:Static d:DataKeys.Booking}}"                                        
                                        CellTemplate="{StaticResource BookingColumnDataTemplate}"
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="120" PrintCellStyle="{StaticResource BookingPrintStyle}"  
                                        VisibleIndex="20"/>
                        <dxg:GridColumn Header="Underlying" FieldName="Underlying" VisibleIndex="30" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="True" UnboundType="String"
                                        CellTemplate="{StaticResource UnderlyingDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource UnderlyingPrintStyle}" />                        
                        
                        <dxg:GridColumn Header="Gross Price" FieldName="GrossPrice" VisibleIndex="40" GridRow="0" HorizontalHeaderContentAlignment="Center" UnboundType="Decimal"
                                        CellTemplate="{StaticResource GrossPriceColumnDataTemplate}"
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="120" PrintCellStyle="{StaticResource GrossPricePrintStyle}"  />
                        <dxg:GridColumn Header="Local Gross Price" FieldName="LocalGrossPrice" VisibleIndex="50" GridRow="0" HorizontalHeaderContentAlignment="Center" UnboundType="Decimal" 
                                        CellTemplate="{StaticResource LocalGrossPriceColumnDataTemplate}"
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="120" PrintCellStyle="{StaticResource LocalGrossPricePrintStyle}" />
                        <dxg:GridColumn FieldName="Quantity" VisibleIndex="60" GridRow="0" HorizontalHeaderContentAlignment="Center" UnboundType="Decimal"
                                        CellTemplate="{StaticResource QuantityColumnDataTemplate}"
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="80" PrintCellStyle="{StaticResource QuantityPrintStyle}" />
                        <dxg:GridColumn Header="Commission %" FieldName="CommissionPercentage" VisibleIndex="70" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="True" UnboundType="Decimal"
                                        CellTemplate="{StaticResource CommissionPercentageDataTemplate}"
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource CommissionPercentagePrintStyle}" />
                        <dxg:GridColumn Header="Ccy" FieldName="SettleCurrency" VisibleIndex="80" GridRow="0" HorizontalHeaderContentAlignment="Center" UnboundType="String"
                                        CellTemplate="{StaticResource SettleCurrencyDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="40" PrintCellStyle="{StaticResource SettleCurrencyPrintStyle}" />
                        <dxg:GridColumn Header="Business Unit" FieldName="BusinessUnit" VisibleIndex="90" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="True" UnboundType="String"
                                        CellTemplate="{StaticResource BusinessUnitDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="88" PrintCellStyle="{StaticResource BusinessUnitPrintStyle}" />
                        <dxg:GridColumn Header="Fund" FieldName="Fund" VisibleIndex="100" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="True" UnboundType="String"
                                        CellTemplate="{StaticResource FundDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="56 " PrintCellStyle="{StaticResource FundPrintStyle}" />
                        <dxg:GridColumn Header="Net Notional" FieldName="NetNotional" VisibleIndex="110" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="True" UnboundType="Decimal" 
                                        CellTemplate="{StaticResource NetNotionalDataTemplate}" 
                                        Width="88" PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" />
                        <dxg:GridColumn Header="Fx Rate" FieldName="FxRate" VisibleIndex="120" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="True" UnboundType="Decimal"
                                        CellTemplate="{StaticResource FxRateDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="72" PrintCellStyle="{StaticResource FxRatePrintStyle}" />
                        <dxg:GridColumn Header="Source System" FieldName="SourceSystem" VisibleIndex="130" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="True" UnboundType="String"
                                        CellTemplate="{StaticResource SourceSystemDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource SourceSystemPrintStyle}" />

                        <dxg:GridColumn Header="Commission %" FieldName="CommissionPercentage" VisibleIndex="140" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="False" UnboundType="Decimal"
                                        CellTemplate="{StaticResource CommissionPercentageDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource CommissionPercentagePrintStyle}" />
                        <dxg:GridColumn Header="Commission per share" FieldName="CommissionAmountPerShare" VisibleIndex="150" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="False" UnboundType="Decimal"
                                        CellTemplate="{StaticResource CommissionAmountPerShareDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource CommissionAmountPerSharePrintStyle}" />
                        <dxg:GridColumn Header="Commission Absolute" FieldName="CommissionAbsolute" VisibleIndex="160" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="False" UnboundType="Decimal"
                                        CellTemplate="{StaticResource CommissionAbsoluteDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource CommissionAbsolutePrintStyle}" />

                        <dxg:GridColumn Header="Research Commission %" FieldName="ResearchCommissionPercentage" VisibleIndex="170" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="False" UnboundType="Decimal"
                                        CellTemplate="{StaticResource ResearchCommissionPercentageDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource ResearchCommissionPercentagePrintStyle}" />
                        <dxg:GridColumn Header="Research Commission per share" FieldName="ResearchCommissionAmountPerShare" VisibleIndex="180" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="False" UnboundType="Decimal" 
                                        CellTemplate="{StaticResource ResearchCommissionAmountPerShareDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource ResearchCommissionAmountPerSharePrintStyle}" />
                        <dxg:GridColumn Header="Research Commission Absolute" FieldName="ResearchCommissionAbsolute" VisibleIndex="190" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="False" UnboundType="Decimal" 
                                        CellTemplate="{StaticResource ResearchCommissionAbsoluteDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource ResearchCommissionAbsolutePrintStyle}" />

                        <dxg:GridColumn Header="Execution Commission %" FieldName="ExecutionCommissionPercentage" VisibleIndex="200" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="False" UnboundType="Decimal" 
                                        CellTemplate="{StaticResource ExecutionCommissionPercentageDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource ExecutionCommissionPercentagePrintStyle}" />
                        <dxg:GridColumn Header="Execution Commission per share" FieldName="ExecutionCommissionAmountPerShare" VisibleIndex="210" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="False" UnboundType="Decimal" 
                                        CellTemplate="{StaticResource ExecutionCommissionAmountPerShareDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource ExecutionCommissionAmountPerSharePrintStyle}" />
                        <dxg:GridColumn Header="Execution Commission Absolute" FieldName="ExecutionCommissionAbsolute" VisibleIndex="220" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="False" UnboundType="Decimal" 
                                        CellTemplate="{StaticResource ExecutionCommissionAbsoluteDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource ExecutionCommissionAbsolutePrintStyle}" />

                        <dxg:GridColumn Header="Market Fees %" FieldName="MarketFeesPercentage" VisibleIndex="230" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="False" UnboundType="Decimal" 
                                        CellTemplate="{StaticResource MarketFeesPercentageDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource MarketFeesPercentagePrintStyle}" />
                        <dxg:GridColumn Header="Market Fees Amount per share" FieldName="MarketFeesAmountPerShare" VisibleIndex="240" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="False" UnboundType="Decimal" 
                                        CellTemplate="{StaticResource MarketFeesAmountPerShareDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource MarketFeesAmountPerSharePrintStyle}" />
                        <dxg:GridColumn Header="Market Fees Absolute" FieldName="MarketFeesAbsolute" VisibleIndex="250" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="False" UnboundType="Decimal" 
                                        CellTemplate="{StaticResource MarketFeesAbsoluteDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource MarketFeesAbsolutePrintStyle}" />

                        <dxg:GridColumn Header="Market Taxes %" FieldName="MarketTaxesPercentage" VisibleIndex="260" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="False" UnboundType="Decimal" 
                                        CellTemplate="{StaticResource MarketTaxesPercentageDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource MarketTaxesPercentagePrintStyle}" />
                        <dxg:GridColumn Header="Market Taxes Amount per share" FieldName="MarketTaxesAmountPerShare" VisibleIndex="270" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="False" UnboundType="Decimal" 
                                        CellTemplate="{StaticResource MarketTaxesAmountPerShareDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource MarketTaxesAmountPerSharePrintStyle}" />
                        <dxg:GridColumn Header="Market Taxes Absolute" FieldName="MarketTaxesAbsolute" VisibleIndex="280" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="False" UnboundType="Decimal" 
                                        CellTemplate="{StaticResource MarketTaxesAbsoluteDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource MarketTaxesAbsolutePrintStyle}" />

                        <dxg:GridColumn Header="Other Fees %" FieldName="OtherFeesPercentage" VisibleIndex="290" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="False" UnboundType="Decimal" 
                                        CellTemplate="{StaticResource OtherFeesPercentageDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource OtherFeesPercentagePrintStyle}" />
                        <dxg:GridColumn Header="Other Fees Amount per share" FieldName="OtherFeesAmountPerShare" VisibleIndex="300" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="False" UnboundType="Decimal" 
                                        CellTemplate="{StaticResource OtherFeesAmountPerShareDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource OtherFeesAmountPerSharePrintStyle}" />
                        <dxg:GridColumn Header="Other Fees Absolute" FieldName="OtherFeesAbsolute" VisibleIndex="310" GridRow="0" HorizontalHeaderContentAlignment="Center" Visible="False" UnboundType="Decimal" 
                                        CellTemplate="{StaticResource OtherFeesAbsoluteDataTemplate}" 
                                        PrintColumnHeaderStyle="{StaticResource DefaultHeaderPrintHeaderStyle}" 
                                        Width="96" PrintCellStyle="{StaticResource OtherFeesAbsolutePrintStyle}" />
                    </dxg:GridControlBand>
                </dxg:GridControl.Bands>
                <dxg:GridControl.View>
                    <dxg:TableView ShowAutoFilterRow="True" FilterRowDelay="800" SearchColumns="Underlying;Quantity" AllowPerPixelScrolling="True" AllowFilterEditor="False"
                                   AllowCascadeUpdate="True" RowAnimationKind="None" ShowIndicator="True" ShowCriteriaInAutoFilterRow="True"
                                   AllowConditionalFormattingMenu="True" CellStyle="{DynamicResource SelectionStateCellStyle}">
                        <i:Interaction.Behaviors>
                            <b:FilterAdjustmentBehaviour/>
                            <b:ContextMenuBehaviour ContextMenus="{Binding ContextMenus}"/>
                        </i:Interaction.Behaviors>
                        <dxg:TableView.RowCellMenuCustomizations>
                            <dxb:BarButtonItem BarItemName="EandP" Content="Exporting and Printing" Glyph="{dx:DXImage Image=Export_16x16.png}">
                                <i:Interaction.Behaviors>
                                    <behaviours1:ExportGridContentsBehavior 
                                        TableToExport="{Binding RelativeSource={RelativeSource AncestorType=dxg:TableView}}" 
                                        OwnerWindow="{Binding RelativeSource={RelativeSource AncestorType=Window}}"/>
                                </i:Interaction.Behaviors>
                            </dxb:BarButtonItem>
                            <dxb:BarItemSeparator />
                        </dxg:TableView.RowCellMenuCustomizations>
                    </dxg:TableView>
                </dxg:GridControl.View>
            </dxg:GridControl>

            <GridSplitter HorizontalAlignment="Stretch" VerticalAlignment="Top" Grid.Row="1" ResizeBehavior="PreviousAndNext" Height="3" Background="{DynamicResource GrayBrush2}"/>

            <dxg:GridControl Grid.Row="2" AllowLiveDataShaping="True" Name="GridControl2" dx:ThemeManager.ThemeName="Office2016WhiteSE" ItemsSource="{Binding Executions}">
                <dxg:GridControl.Bands>
                    <dxg:GridControlBand Header="Band1" VisibleIndex="0" OverlayHeaderByChildren="True">
                        <dxg:GridColumn x:Name="Execution_Type" Header="Type" FieldName="Type" HorizontalHeaderContentAlignment="Center" VisibleIndex="1"/>
                        <dxg:GridColumn x:Name="Execution_LinkageId" Header="Linkage Id"  FieldName="LinkageId" HorizontalHeaderContentAlignment="Center" Visible="False" VisibleIndex="2"/>
                        <dxg:GridColumn x:Name="Execution_RowId" Header="Row Id"  FieldName="RowId" HorizontalHeaderContentAlignment="Center"  Visible="False" VisibleIndex="3"/>
                        <dxg:GridColumn x:Name="Execution_LocalGrossPrice" Header="Local Gross Price" FieldName="LocalGrossPrice"                                         
                                    HorizontalHeaderContentAlignment="Center" ReadOnly="True" CellToolTipBinding="{Binding RowData.Row.LocalGrossPrice}" VisibleIndex="4">
                            <dxg:GridColumn.EditSettings>
                                <dxe:TextEditSettings DisplayFormat="{x:Static Member=inf:StringFormats.CustomCellDecimalPrecision}"></dxe:TextEditSettings>
                            </dxg:GridColumn.EditSettings>
                        </dxg:GridColumn>
                        <dxg:GridColumn x:Name="Execution_GrossPrice" Header="Gross Price" FieldName="GrossPrice" 
                                        HorizontalHeaderContentAlignment="Center" ReadOnly="True" CellToolTipBinding="{Binding RowData.Row.GrossPrice}" VisibleIndex="5">
                            <dxg:GridColumn.EditSettings>
                                <dxe:TextEditSettings DisplayFormat="{x:Static Member=inf:StringFormats.CustomCellDecimalPrecision}"></dxe:TextEditSettings>
                            </dxg:GridColumn.EditSettings>
                        </dxg:GridColumn>
                        <dxg:GridColumn x:Name="Execution_NetNotional" Header="Net Notional" FieldName="NetNotional" 
                                        HorizontalHeaderContentAlignment="Center" ReadOnly="True" CellToolTipBinding="{Binding RowData.Row.NetNotional}" VisibleIndex="6">
                            <dxg:GridColumn.EditSettings>
                                <dxe:TextEditSettings 
                                HorizontalContentAlignment="Right" 
                                DisplayFormat="{x:Static Member=inf:StringFormats.CustomCellDecimalPrecision}"></dxe:TextEditSettings>
                            </dxg:GridColumn.EditSettings>
                        </dxg:GridColumn>

                        <dxg:GridColumn x:Name="Execution_Quantity" Header="Quantity" FieldName="Quantity" 
                                        HorizontalHeaderContentAlignment="Center" ReadOnly="True" CellToolTipBinding="{Binding RowData.Row.Quantity}" VisibleIndex="7">
                            <dxg:GridColumn.EditSettings>
                                <dxe:TextEditSettings HorizontalContentAlignment="Right" 
                                                  DisplayFormat="{x:Static Member=inf:StringFormats.CustomCellLongPrecision}"></dxe:TextEditSettings>
                            </dxg:GridColumn.EditSettings>
                        </dxg:GridColumn>
                        <dxg:GridColumn x:Name="Execution_SettleCurrency" Header="Settle Currency" FieldName="SettleCurrency" HorizontalHeaderContentAlignment="Center" VisibleIndex="8" />
                        <dxg:GridColumn x:Name="Execution_SourceSystem" Header="Source System" FieldName="Source System" HorizontalHeaderContentAlignment="Center" VisibleIndex="9" />
                        <dxg:GridColumn x:Name="Execution_CommissionPercentage" Header="CommissionPercentage" FieldName="CommissionPercentage" 
                                        HorizontalHeaderContentAlignment="Center" ReadOnly="True" Visible="False" 
                                        CellToolTipBinding="{Binding RowData.Row.CommissionPercentage}" VisibleIndex="10">
                            <dxg:GridColumn.EditSettings>
                                <dxe:TextEditSettings HorizontalContentAlignment="Right" 
                                                  DisplayFormat="{x:Static Member=inf:StringFormats.CustomPercentagePrecision}"></dxe:TextEditSettings>
                            </dxg:GridColumn.EditSettings>
                        </dxg:GridColumn>
                        <dxg:GridColumn x:Name="Execution_CommissionAmountPerShare" Header="CommissionAmountPerShare" FieldName="CommissionAmountPerShare" 
                                        HorizontalHeaderContentAlignment="Center" ReadOnly="True" Visible="False" 
                                        CellToolTipBinding="{Binding RowData.Row.CommissionAmountPerShare}" VisibleIndex="11">
                            <dxg:GridColumn.EditSettings>
                                <dxe:TextEditSettings HorizontalContentAlignment="Right" 
                                                  DisplayFormat="{x:Static Member=inf:StringFormats.CustomCellDecimalPrecision}"></dxe:TextEditSettings>
                            </dxg:GridColumn.EditSettings>
                        </dxg:GridColumn>
                        <dxg:GridColumn x:Name="Execution_CommissionAbsolute" Header="Source System" FieldName="CommissionAbsolute" 
                                        HorizontalHeaderContentAlignment="Center" ReadOnly="True" Visible="False" 
                                        CellToolTipBinding="{Binding RowData.Row.CommissionAmountPerShare}" VisibleIndex="12">
                            <dxg:GridColumn.EditSettings>
                                <dxe:TextEditSettings HorizontalContentAlignment="Right" 
                                                  DisplayFormat="{x:Static Member=inf:StringFormats.CustomCellDecimalPrecision}"></dxe:TextEditSettings>
                            </dxg:GridColumn.EditSettings>
                        </dxg:GridColumn>
                    </dxg:GridControlBand>
                </dxg:GridControl.Bands>
                <dxg:GridControl.View>
                    <dxg:TableView ShowIndicator="False"/>
                </dxg:GridControl.View>
                <dxg:GridControl.Style>
                    <Style TargetType="{x:Type dxg:GridControl}">
                        <Style.Triggers>
                            <DataTrigger Binding="{Binding SelectedDetail}" Value="{x:Null}">
                                <Setter Property="Visibility" Value="Collapsed"/>
                            </DataTrigger>
                            <DataTrigger Binding="{Binding SelectedDetail.Type}" Value="ALS">
                                <Setter Property="Visibility" Value="Collapsed"/>
                            </DataTrigger>
                            <DataTrigger Binding="{Binding SelectedDetail.Type}" Value="AL">
                                <Setter Property="Visibility" Value="Collapsed"/>
                            </DataTrigger>
                        </Style.Triggers>
                    </Style>
                </dxg:GridControl.Style>
            </dxg:GridControl>

            <dxg:GridControl Grid.Row="2" AllowLiveDataShaping="True" Name="GridControl3" dx:ThemeManager.ThemeName="Office2016WhiteSE" ItemsSource="{Binding AllocationRequests}">
                <dxg:GridControl.Bands>
                    <dxg:GridControlBand Name="band1" VisibleIndex="0" OverlayHeaderByChildren="True">
                        <dxg:GridColumn x:Name="AllocationRequest_Type" Header="Type" FieldName="Type" HorizontalHeaderContentAlignment="Center"  VisibleIndex="1" />
                        <dxg:GridColumn x:Name="AllocationRequest_LinkageId" Header="Linkage Id"  FieldName="LinkageId" HorizontalHeaderContentAlignment="Center" Visible="False" VisibleIndex="2"/>
                        <dxg:GridColumn x:Name="AllocationRequest_RowId" Header="Row Id"  FieldName="RowId" HorizontalHeaderContentAlignment="Center"  Visible="False" VisibleIndex="3"/>
                        <dxg:GridColumn x:Name="AllocationRequest_AllocationRowId" Header="Allocation Row Id"  FieldName="AllocationRowId" HorizontalHeaderContentAlignment="Center"  
                                        Visible="False" VisibleIndex="4"/>
                        <dxg:GridColumn x:Name="AllocationRequest_LocalGrossPrice" Header="Local Gross Price" FieldName="LocalGrossPrice"                                         
                                        HorizontalHeaderContentAlignment="Center" ReadOnly="True" 
                                        CellToolTipBinding="{Binding RowData.Row.LocalGrossPrice}" VisibleIndex="5">
                            <dxg:GridColumn.EditSettings>
                                <dxe:TextEditSettings DisplayFormat="{x:Static Member=inf:StringFormats.CustomCellDecimalPrecision}"></dxe:TextEditSettings>
                            </dxg:GridColumn.EditSettings>
                        </dxg:GridColumn>
                        <dxg:GridColumn x:Name="AllocationRequest_GrossPrice" Header="Gross Price" FieldName="GrossPrice" 
                                        HorizontalHeaderContentAlignment="Center" ReadOnly="True" 
                                        CellToolTipBinding="{Binding RowData.Row.GrossPrice}" VisibleIndex="6">
                            <dxg:GridColumn.EditSettings>
                                <dxe:TextEditSettings DisplayFormat="{x:Static Member=inf:StringFormats.CustomCellDecimalPrecision}"></dxe:TextEditSettings>
                            </dxg:GridColumn.EditSettings>
                        </dxg:GridColumn>
                        <dxg:GridColumn x:Name="AllocationRequest_NetNotional" Header="Net Notional" FieldName="NetNotional" 
                                        HorizontalHeaderContentAlignment="Center" ReadOnly="True" 
                                        CellToolTipBinding="{Binding RowData.Row.NetNotional}" VisibleIndex="7">
                            <dxg:GridColumn.EditSettings>
                                <dxe:TextEditSettings 
                                    HorizontalContentAlignment="Right" 
                                    DisplayFormat="{x:Static Member=inf:StringFormats.CustomCellDecimalPrecision}"></dxe:TextEditSettings>
                            </dxg:GridColumn.EditSettings>
                        </dxg:GridColumn>
                        <dxg:GridColumn x:Name="AllocationRequest_Quantity" Header="Quantity" FieldName="Quantity" 
                                        HorizontalHeaderContentAlignment="Center" ReadOnly="True" 
                                        CellToolTipBinding="{Binding RowData.Row.Quantity}" VisibleIndex="8">
                            <dxg:GridColumn.EditSettings>
                                <dxe:TextEditSettings HorizontalContentAlignment="Right" 
                                                      DisplayFormat="{x:Static Member=inf:StringFormats.CustomCellLongPrecision}"></dxe:TextEditSettings>
                            </dxg:GridColumn.EditSettings>
                        </dxg:GridColumn>
                        <dxg:GridColumn x:Name="AllocationRequest_SettleCurrency" Header="Settle Currency" FieldName="SettleCurrency" 
                                        HorizontalHeaderContentAlignment="Center" VisibleIndex="9" />
                        <dxg:GridColumn x:Name="AllocationRequest_SourceSystem" Header="Source System" FieldName="SourceSystem" 
                                        HorizontalHeaderContentAlignment="Center" VisibleIndex="10" />
                        <dxg:GridColumn x:Name="AllocationRequest_CommissionPercentage" Header="CommissionPercentage" FieldName="CommissionPercentage" 
                                        HorizontalHeaderContentAlignment="Center" ReadOnly="True" Visible="False" 
                                        CellToolTipBinding="{Binding RowData.Row.CustomPercentagePrecision}" VisibleIndex="11">
                            <dxg:GridColumn.EditSettings>
                                <dxe:TextEditSettings HorizontalContentAlignment="Right" 
                                                      DisplayFormat="{x:Static Member=inf:StringFormats.CustomPercentagePrecision}"></dxe:TextEditSettings>
                            </dxg:GridColumn.EditSettings>
                        </dxg:GridColumn>
                        <dxg:GridColumn x:Name="AllocationRequest_CommissionAmountPerShare" Header="CommissionAmountPerShare" FieldName="CommissionAmountPerShare" 
                                        HorizontalHeaderContentAlignment="Center" ReadOnly="True" Visible="False" 
                                        CellToolTipBinding="{Binding RowData.Row.CommissionAmountPerShare}"
                                        VisibleIndex="12">
                            <dxg:GridColumn.EditSettings>
                                <dxe:TextEditSettings HorizontalContentAlignment="Right" 
                                                      DisplayFormat="{x:Static Member=inf:StringFormats.CustomCellDecimalPrecision}"></dxe:TextEditSettings>
                            </dxg:GridColumn.EditSettings>
                        </dxg:GridColumn>
                        <dxg:GridColumn x:Name="AllocationRequest_CommissionAbsolute" Header="Source System" FieldName="CommissionAbsolute" 
                                        HorizontalHeaderContentAlignment="Center" ReadOnly="True" Visible="False" 
                                        CellToolTipBinding="{Binding RowData.Row.CommissionAbsolute}"
                                        VisibleIndex="13">
                            <dxg:GridColumn.EditSettings>
                                <dxe:TextEditSettings HorizontalContentAlignment="Right" 
                                                      DisplayFormat="{x:Static Member=inf:StringFormats.CustomCellDecimalPrecision}"></dxe:TextEditSettings>
                            </dxg:GridColumn.EditSettings>
                        </dxg:GridColumn>
                    </dxg:GridControlBand>
                </dxg:GridControl.Bands>
                <dxg:GridControl.View>
                    <dxg:TableView ShowIndicator="False"/>
                </dxg:GridControl.View>
                <dxg:GridControl.Style>
                    <Style TargetType="{x:Type dxg:GridControl}">
                        <Style.Triggers>
                            <DataTrigger Binding="{Binding SelectedDetail}" Value="{x:Null}">
                                <Setter Property="Visibility" Value="Collapsed"/>
                            </DataTrigger>
                            <DataTrigger Binding="{Binding SelectedDetail.Type}" Value="BLS">
                                <Setter Property="Visibility" Value="Collapsed"/>
                            </DataTrigger>
                            <DataTrigger Binding="{Binding SelectedDetail.Type}" Value="BL">
                                <Setter Property="Visibility" Value="Collapsed"/>
                            </DataTrigger>
                        </Style.Triggers>
                    </Style>
                </dxg:GridControl.Style>
            </dxg:GridControl>

            <Grid Grid.Row="0" Grid.RowSpan="3" 
                  Visibility="{Binding IsLoading, Converter={StaticResource BooleanToVisibilityConverter}}" 
                  Background="{DynamicResource GrayBrush2}">
                <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">
                    <c:ProgressRing IsActive="{Binding IsLoading}"/>
                    <TextBlock Text="Won't be a moment, just loading your data."/>
                </StackPanel>
            </Grid>
        </Grid>
    </DataTemplate>

</ResourceDictionary>



