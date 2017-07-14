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
